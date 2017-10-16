---
layout: post
category: code reading
title: IJCAI 2017 OOKB (main.py)
tagline: by wencolani
tags: [code reading, paper code]
---

* paper : Knowledge Transfer for Out-of-Knowledge-Base Entities: A Graph Neural Network Approach
* code from : https://github.com/takuo-h/GNN-for-OOKB

```python

# -*- coding: utf-8 -*-

import os
os.environ['CHAINER_TYPE_CHECK'] = '0'

# wen: chianer is 'a powerful flexible, and intuitive framework for Neural Network'
import chainer 

# wen: cupy is a 'NumPy-like API accelerate with CUDA'	
import cupy as cp 	
import numpy as np 
from chainer import serializers

# wen: in this file, there are a lot of usually used function defined 
import util.general_tool as tool 	
from util.backend import XP
from util.vocabulary import Vocabulary
from util.optimizer_manager import get_opt
from models.manager import get_model

#----------------------------------------------------------------------------
from collections import defaultdict

# wen: python collection.defaultdict is a very useful datastructure,
# you could assign the datastructure of the values in defaultdict when you creat it
candidate_heads=defaultdict(set)  

# wen: these are globle parameters			  
gold_heads = defaultdict(set)

candidate_tails=defaultdict(set)
gold_tails = defaultdict(set)
black_set = set()

tail_per_head=defaultdict(set)
head_per_tail=defaultdict(set)

train_data,dev_data,test_data=[],[],[]

trfreq = defaultdict(int)

"""
positive/negativeを返すように
"""
from more_itertools import chunked
import random
def gen_batch(args,mode='train'):
	# wen: use them as global parameters, only read the train/test/dev file once
	global train_data,dev_data,test_data,trfreq 

	if len(train_data)==0:
		train_data = set()
		for line in open(args.train_file):
			items = list(map(int,line.strip().split('\t')))
			if len(items)==4:
				h,r,t,l = items
				if l==0: continue
			else:
				h,r,t = items
			train_data.add((h,r,t,))
			trfreq[r]+=1
		train_data = list(train_data)
		for r in trfreq:
			# wen: what is [trfreq[r]] defined for?
			trfreq[r] = args.train_size/(float(trfreq[r])*len(trfreq)) 

	if len(dev_data)==0:
		dev_data = set()
		for line in open(args.dev_file):
			h,r,t,l = list(map(int,line.strip().split('\t')))
			dev_data.add((h,r,t,l,))
		dev_data=list(dev_data)
		print('dev size:',len(dev_data))
	if len(test_data)==0:
		# wen: read file like this seems more elegent
		test_data = [line.strip().split('\t') for line in open(args.test_file)] 
		test_data = [tuple(map(int,x)) for x in test_data]
		test_data = list(set(test_data))
		print('test size:',len(test_data))

	# wen: 不是很明白这种检测性的代码存在的意义
	if mode not in ['train','dev','test']:  
		print('no such mode in gen_data',mode)
		sys.exit(1)
	elif mode=='train':
		# wen: what is the difference between [args.train_size] and [args.batch_size]
		skip_rate = args.train_size/float(len(train_data))  

		positive,negative=[],[]

		# wen: i like random.shuffle() 
		random.shuffle(train_data)  

		for i in range(len(train_data)):
			h,r,t = train_data[i]
			# ignore the triple exist in [blace_set]
			if (-r,t) in black_set: continue 
			if (h, r) in black_set: continue

			if args.is_balanced_tr:
				if random.random()>trfreq[r]: continue
			else:
				if random.random()>skip_rate: continue

			# tph/Z
			head_ratio = 0.5  # wen: unif setting
			if args.is_bernoulli_trick: # wen: bern setting
				head_ratio = tail_per_head[h]/(tail_per_head[h]+head_per_tail[t])
			if random.random()>head_ratio:
				cand = random.choice(candidate_heads[r])
				while cand in gold_heads[(r,t)]:
					# wen: this is a small trick: to generat negative triple by replace head/tail
					# wen: with the entities that are once act as a head or tail entity in triple contained relation r
					cand = random.choice(candidate_heads[r])  
															  
				h = cand
			else:
				cand = random.choice(candidate_tails[r])
				while cand in gold_tails[(h,r)]:
					cand = random.choice(candidate_tails[r])
				t = cand
			# wen: gerate one negtive triple for every positive triple
			if len(positive)==0 or len(positive) <= args.batch_size:  
				positive.append(train_data[i])
				negative.append((h,r,t))
			else:
				# wen: yield positve truples and negative triples by batch_size
				yield positive,negative 
				# wen: if the len of positive triples meet batch_size then current triples pair were not add 
				# wen: so need to set positive,negative = [train_data[i]],[(h,r,t)]
				positive,negative = [train_data[i]],[(h,r,t)]  
		# wen: this is set for the last batch in which the length of [positve] is likely to less than batch_size									   
		if len(positive)!=0:  
			yield positive,negative
	elif mode=='dev':
		# wen: there is no module is called more_itertools and what is function chuncked()?
		# wen: chuncked() will yeild triple by the input batch_size
		for batch in chunked(dev_data, args.test_batch_size): 
															  
			yield batch
	elif mode=='test':
		for batch in chunked(test_data, args.test_batch_size):
			yield batch

#----------------------------------------------------------------------------

def train(args,m,xp,opt,links,relations,edges):
	Loss,N = [],0
	for positive,negative in gen_batch(args,mode='train'):
		loss = m(positive,negative,links,relations,edges,xp,True)
		loss.backward()
		opt.update()
		Loss.append(float(loss.data)/len(positive))
		N += len(positive)
		del loss
	return sum(Loss),N

def evaluate(args,m,xp,links,relations,edges):
	for mode in ['dev','test']:
		values = []
		for batch in gen_batch(args,mode=mode):
			margins = m.get_margins(batch,links,relations,edges,xp,mode)
			for v,(h,r,t,l) in zip(margins.data,batch):
				values.append((h,r,t,l,v))
			del margins
		if args.margin_file!='':
			with open(args.margin_file,'a') as wf:
				wf.write(mode+':'+' '.join([','.join(list(map(str,x))) for x in values])+'\n')

def get_sizes(args):
	"""
	relation,entity=set(),set()
	for l in open(args.train_file):
		h,r,t,l = l.strip().split('\t')
		entity.add(h)
		relation.add(r)
		entity.add(t)
	return len(relation),len(entity)
	"""
	relation,entity=-1,-1
	for line in open(args.train_file):
		# wen: apply int() function for every element input_list by map()
		# wen: the blueprint of map() is: map(function_to_apply, list_of_inputs)
		items = list(map(int,line.strip().split('\t'))) 
														
		if len(items)==4:
			# wen: what is l ?
			h,r,t,l = items 

			if l==0: continue
		else:
			h,r,t = items
		relation = max(relation,r)
		entity = max(entity,h,t)
	return relation+1,entity+1 
	# wen:这种get_sizes的方法真是清奇, 相比起上面作者注释掉的方法，这个的计算量会小很多，
	# wen:但前提是每个entity和relation都已经分配了连续的id
								

import sys
def main(args):
	global candidate_heads,gold_heads,candidate_tails,gold_tails,black_set
	xp = XP(args)

	# wen: if the args.rel_size and args.entity_size is set by function get_size()
	# wen: then it is better not add them with P.add_argument(). only the parameters deployable are needed to be added
	args.rel_size,args.entity_size = get_sizes(args) 
													 
	print('relation size:',args.rel_size,'entity size:',args.entity_size)
	m = get_model(args)
	opt = get_opt(args)
	opt.setup(m)

	relations = dict()
	links = defaultdict(set)
	for line in tool.read(args.train_file):
		items = list(map(int,line.strip().split('\t')))
		if len(items)==4:
			h,r,t,l = items
			if l==0: continue
		else:
			h,r,t = items

		# wen: relations:{(h1,t1):r1, (h2,t2):r2,...}
		relations[(h,t)]=r 

		# wen: links:(defaultdict){e1:[e2, e3, e5...], e2:{e2,e4, e8,...}}. neglect the direction of relation
		links[t].add(h)    
		links[h].add(t)

		# wen: [gold_heads] is defaultdict and a global parameter and record all head entities connecting to for every (r,t)
		gold_heads[(r,t)].add(h)

		# wen: [gold_tails] record all tail entities connecting to for every (h,r) 
		gold_tails[(h,r)].add(t)

		# wen: [candidate_heads]: record all head entities for relation r 
		candidate_heads[r].add(h)

		# wen: [candidate_tails]: record all tial entities for relation r 
		candidate_tails[r].add(t)

		# wen: [tail_per_head]: all the tail entities t connected to head entity h 
		tail_per_head[h].add(t)

		# wen: [head_per_tail:] all the head entitise h connected to tail entity t
		# wen: [links] is a combination of [tail_per_head] and [head_per_tail] 
		head_per_tail[t].add(h) 
							    
	for e in links:
		# wen: the default value of keys in defaultdict is list
		# wen: but we could also set it with other datatype like this: links = defaultdict(set) 
		links[e] = list(links[e]) 
								  

	for p in gold_heads:
		if len(candidate_heads[p[0]]-gold_heads[p])==0:
			p = (-p[0],p[1])

			# wen: [black_set] stores the relation that only connecting to one head or tail entity
			black_set.add(p)

	for p in gold_tails:
		if len(candidate_tails[p[1]]-gold_tails[p])==0:
			black_set.add(p)
	print('black list size:',len(black_set))	
	for r in candidate_heads:
		candidate_heads[r] = list(candidate_heads[r])
	for r in candidate_tails:
		candidate_tails[r] = list(candidate_tails[r])
	for h in tail_per_head:
		# wen: use +0.0 to conver the number from int to float
		tail_per_head[h] = len(tail_per_head[h])+0.0
	for t in head_per_tail:
		head_per_tail[t] = len(head_per_tail[t])+0.0

	if args.train_file==args.auxiliary_file:
		tool.trace('use: edges=links')
		edges = links
	else:
		tool.trace('use: different edges')
		edges = defaultdict(set)

		# wen: in tool.read() will yeild line.strip() for every line in the file
		for line in tool.read(args.auxiliary_file):
			# wen: there is duplicated strip()   
			items = list(map(int,line.strip().split('\t')))  
			if len(items)==4:
				h,r,t,l = items
				if l==0: continue
			else:
				h,r,t = items
			relations[(h,t)]=r
			edges[t].add(h)
			edges[h].add(t)
		for e in edges:
			# wen: what is the differece between (use: edges = links) and (use: different edges)?
			edges[e] = list(edges[e])  

	for epoch in range(args.epoch_size):
		opt.alpha = args.beta0/(1.0+args.beta1*epoch)
		trLoss,Ntr = train(args,m,xp,opt,links,relations,edges)
		evaluate(args,m,xp,links,relations,edges)
		tool.trace('epoch:',epoch,'tr Loss:',tool.dress(trLoss),Ntr)

#----------------------------------------------------------------------------

import sys
def test(args):
	if args.param_dir=='': return

#----------------------------------------------------------------------------
"""
	-tF dataset/data/Freebase13/train \
	-dF dataset/data/Freebase13/train \
	-eF dataset/data/Freebase13/test \
"""

# wen: "the argparse model makes it easy to write user-friendly command-line interface.
# The program defines what the arguments it requires, and argparse will figure out how
# parse those out of sys.argv. The argparse module also automatically generates help and
# usage messages and issues error when users give the program invalid arguments."  
from argparse import ArgumentParser 	
										
										
										
def argument():
	p = ArgumentParser()

	# wen: add_argument('--commandline parameter', 'name for the parameter used in the code', default= default value,
	# wen:  type= the type for this parameter,  )
	p.add_argument('--train_file',      '-tF',  default='dataset/data/Freebase13/classify/train')  
																								   
	p.add_argument('--dev_file',        '-vF',  default='dataset/data/Freebase13/classify/dev')
	p.add_argument('--test_file',       '-eF',  default='dataset/data/Freebase13/classify/test')
	p.add_argument('--auxiliary_file',  '-aF',  default='dataset/data/Freebase13/classify/train')
	p.add_argument('--param_dir',       '-pD',  default='')
	p.add_argument('--margin_file',     '-mF',  default='')

	p.add_argument('--use_gpu',     '-g',   default=False,  action='store_true')
	p.add_argument('--gpu_device',  '-gd',  default=0,      type=int)

	p.add_argument('--seed',        '-seed',default=0,      type=int)

	p.add_argument('--rel_size',  '-Rs',       default=10,    type=int)
	p.add_argument('--entity_size', '-Es',       default=18,    type=int)

	p.add_argument('--is_residual',     '-iR',   default=False,  action='store_true')
	p.add_argument('--is_batchnorm',    '-iBN',  default=False,  action='store_true')
	p.add_argument('--is_embed',      	'-nE',   default=True,   action='store_false')
	p.add_argument('--is_known',    	'-nK',   default=True,   action='store_false')
	p.add_argument('--is_balanced_tr',    '-nBtr',   default=True,   action='store_false')
	p.add_argument('--is_balanced_dev',   '-iBde',   default=False,   action='store_true')
	p.add_argument('--is_bernoulli_trick',  '-iBeT',   default=False,   action='store_true')

	p.add_argument('--is_bound_wr',   '-iRB',   default=False,   action='store_true')
	p.add_argument('--object_kind',   '-oK',    default=1,  type=int)

	p.add_argument('--layerR' ,      '-Lr',      default=1,      type=int)

	p.add_argument('--dim',         '-D',       default=200,    type=int)
	p.add_argument('--order',       '-O',       default=1,      type=int)
	p.add_argument('--threshold',   '-T',       default=1.0,   type=float)

	p.add_argument('--dropout_block','-dBR',     default=0.05,  type=float)
	p.add_argument('--dropout_decay','-dDR',     default=0.0,   type=float)
	p.add_argument('--dropout_embed','-dER',     default=0.0,   type=float)

	p.add_argument('--train_size',  '-trS',  default=1000,       type=int)

	p.add_argument('--batch_size',  '-bS',  default=64,        type=int)
	p.add_argument('--test_batch_size',  '-tbS',  default=64,        type=int)
	p.add_argument('--sample_size', '-sS',  default=64,        type=int)
	p.add_argument('--pool_size',   '-pS',  default=128*5,      type=int)
	p.add_argument('--epoch_size',  '-eS',  default=1000,       type=int)
	p.add_argument('--nn_model',    '-nn',  default='H')
	p.add_argument('--activate',    '-af',  default='tanh')
	p.add_argument('--pooling_method',    '-pM',  default='max')

	p.add_argument('--opt_model',   "-Op",  default="Adam")
	p.add_argument('--alpha0',      "-a0",  default=0,      type=float)
	p.add_argument('--alpha1',      "-a1",  default=0,      type=float)
	p.add_argument('--alpha2',      "-a2",  default=0,      type=float)
	p.add_argument('--alpha3',      "-a3",  default=0,      type=float)
	p.add_argument('--beta0',       "-b0",  default=0.01,   type=float)
	p.add_argument('--beta1',       "-b1",  default=0.001,  type=float)

	args = p.parse_args()
	return args

import random
if __name__ == '__main__':
	args = argument()
	print(args)

	# wen: sys.argv will return a list of command line arguments passed to a Python script. 
	print(' '.join(sys.argv)) 

	# wen: check all the paths is exit
	tool.check_exists(args.auxiliary_file,args.auxiliary_file,args.param_dir) 
	if args.seed!=-1:
		random.seed(args.seed)
		np.random.seed(args.seed)

		# wen: once you put the same seed you will get the same pattern of the random number
		cp.random.seed(args.seed) 
			
	main(args)


```


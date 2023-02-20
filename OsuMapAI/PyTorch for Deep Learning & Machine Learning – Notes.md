### GQnotes:
1. [QuickLink](https://www.youtube.com/watch?v=V_xro1bcAuA)GQ. [Lerarn PyTorch for Deep Learning](https://www.learnpytorch.io/), 
	- Machine learning is to finding patterns from number described many cases (large data set) 
	- AI is bigest set, subset of this is ML, subset of this is DL. 
	- We have Unstructured data (voice analysis), so we need DeepLearning instead of MachineLearning
		- example algorithms are:
			- Neural networks
			- Fully connected neural network
			- Convolutional neural network
			- Recurrent neural network
			- Transformer
			- ...many more
		- first three is the topic of this course (with PyTorch)
	- [ends](https://youtu.be/V_xro1bcAuA?t=1402) at 23:20 speed 2.2x is ok, can be faster
2. More therory about deep learning
	1. Types of learning: 
		- Supervised
		- Unsupervised & Self-supervised
		- Transfer Learning
	
		 We need first of these three. When we will have a model, we can use third option to improve
		
		Deep Learning Use Cases -> Translation or Speech recognition -> Is called Sequence to sequence, shorter #seq2seq . 
		[PyTorch](https://pytorch.org/) this page should be visited most. PyTorch is deep learning framework. Is open source and most popular. OpenAI uses  PyTorch.  Is absolutely everywhere. Supports ussage of GPU/TPU. We use tensors as a representation processing data input/output
		- [ends](https://youtu.be/V_xro1bcAuA?t=3835) at 1:03:55 speed 2.2x is ok,
3. Adveces for course instructor:
	1. How to approach course:
		1. Code along
		2. Explore and experiment
		3. Vizualize what you don't understand
		4. Ask questions
		5. Do the exercises
		6. Share your work
	2. How not approach this course 
		1. Avoid work, no make you brain overheat because PyTorch is on fire (torch :))
	3. Learning sources: 
		1. [Course materials](https://www.github.com/mrdbourke/pytorch-deep-learning)
		2. [Course Q&A](https://www.github.com/mrdbourke/pytorch-deep-learning/discussions)
		3. [Course online book](https://learnpytorch.io)
		4. [PyTorch website & forums](https://pytorch.org/)
	4. For course sharing and work use [Google Colab](https://colab.research.google.com/) 
		1. What i Colab, maby worth to check it in future. Becourse this is text editor. But it  more like a IDE with Python compiler and instaled PyTorch. 
			 - [ends](https://youtu.be/V_xro1bcAuA?t=4879) at 1:21:19 speed 2.2x is ok,
4.  Code part
	1. [Colab](https://colab.research.google.com/) , [PyTorch Fundamentals](https://www.learnpytorch.io/00_pytorch_fundamentals/), first add name for the file in Colab, click connect (in the line where the Code and Text are but on the right side).  You can write command and click the play button. For example: print("this is my first run course of PyTorch!")
	2. You can also check Runtime -> change Runtime -> select Hardware accelerator for GPU. Is important especially for Deep learning. You can check access to a GPU by type: !nvidia-smi and run. 
	3. You can write code as a Text with lines exacelly as in code. And you can change it to text by using cmd+m shortcut.
	4. Colab can import libraries. So you can write
		1. import torch
		2. print(torch.__version__)
			and get the resoult: 1.13.1+cu116, tat mean you have PyTorch 1.13.1 and [CUDA Toolkit](https://www.google.com/search?client=firefox-b-d&q=cludatoolkit) version 116. Cuda is this what allows you to run PyTorch on GPU.
	5. Introduction to Tensors. 
		1. PyTorch tensors are created using ['torch.Tensor()'](https://pytorch.org/docs/stable/tensors.html)
		2. Scalar, Vector, Matrix, Tensor -> we make it by adding more square brackets
		3. usefull functions/commands -> TENSOR.ndim, TENSOR.shape, TENSOR[0], 
		4.  [ends](https://youtu.be/V_xro1bcAuA?t=5734) at 1:35:34 speed 1.7x is slow down, becouse of practice mode. Is better to practice with lecturer. Is very important do do the code by yourself. 
	
____
TODO: note: Correct this notes as a practicie for learn english
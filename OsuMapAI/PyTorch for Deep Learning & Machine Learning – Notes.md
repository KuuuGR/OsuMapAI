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
	6. Tensors continue <eng.to.correct>
		1. Naming: scalar, vector -> lowercased, matrix, tensor -> uppercased
		2. Random tensors. 
			- Neural networks learn starts with tensor full of random numbers and then adjust those those to better represent the dat. Start with random numbers -> look at data -> update random numbers -> look at data -> update random numbers
			- colab have smart competion, but is good to go to documentation  [TORCH.RAND](https://pytorch.org/docs/stable/generated/torch.rand.html). Example: torch.rand(4), torch.rand(2, 3)
			- Create a random tensor with similar shape to an image tensor: 
				- random_image_size_tensor = torch.rand(size =(224, 224, 3)) //height, width, colour channels (R, G, B)
				- random_image_size_tensor.shape, random_image_size_tensor.ndim
			-  [ends](https://youtu.be/V_xro1bcAuA?t=6334) at 1:45:34 speed 1.7x is slow down, becouse of practice mode. Is better to understand
	7.  Continue with tensors:
		1. Create tensors with all zeros
			1. zeros = torch.zeros(size=(3,4))
			2. zeros
		2. Create tensors with all ones
			1. ones = torch.ones(size=(3,4))
			2. ones
		3. Somthing about data type
			1. ones.dtype
			2. zreos.dtype
			3. torch.float.32
		4. Creating a range of tensors and tentors-like
			1. torch.range(0, 10) //still working but is deprecated and will be removed
			2. torch.arange(1, 10)
		5. To look at the documentation press shift + tab
			1. Documentation of [pytorch.arange](https://pytorch.org/docs/stable/generated/torch.arange.html)
			2. ten_zeros = torch.zeros_like(input=one_to_ten) //its like method create tesor with same shape
			3. //sometimes it works so slow or even hand it need to choose restart and runtime all form upper menu
		4.  [ends](https://youtu.be/V_xro1bcAuA?t=6838) at 1:53:58 speed 1.7x is slow down, becouse of practice mode. Is better to understand
		6. Work with DataTypes precision in computing, 
			1. dtype, device, requires_grad
		7. Tensor operations [MatrixMultiplication cool animation](http://matrixmultiplication.xyz/)
			1. matrix multiplication matmul(...), also can use @, or alias mm(...)
			2. transpose tensor tensor_B.T.shape
			3. tensor aggregatin (find the min, max, mean, sum), 
				1. x.argmin() find the position where minimum value is
				2. x.argmax()
			4. Reshaping, stacking, squeezing, and unsqueezing tensors
				1. Reshaping - reshapes an input tensor to a defined shape
				2. View - Return a view of an input tensor of certain shape but keep the same memory as the oryginal tensor
				3. Stacking - combine multiple tensors on top of each other (vstack) or side by side (hstack)
				4. Squeeze - removes all '1' dimensions from a tensor
				5. Unsqueeze - add a '1' dimension to a target tensor
				6. Permute - Return a view of the input with dimensions permuted (swapped) in a certain way
				7.  - [ends](https://youtu.be/V_xro1bcAuA?t=11496) at 3:11:36 speed 2.2x is ok, because is all repetition from math course
			8. More math. Maby worth to come back. All about tensors acces and tenstor modyfication. PyCharm and PyTorch stuff. Next topic is a Reproducbility - somthing about ronadom stuff
				1. Reduce randomness. Use seed does is 'flavour'. 
					RANDOM_SEED = 42
					 torch_manual_seed(RANDOM_SEED)
				2. Runing tensors and PyTorch objects on the GPUs (and making things faster stuff)
				3. Check for GPU access with PyTorch. Set device if GPU access for CPU when is no GPU device. Assign and unassign GPU to CPU and CPU to GPU
				4. end 32.PyTorch Fundamentals exercises & extra-curriculum
				   [ends](https://youtu.be/V_xro1bcAuA?t=15159) at 4:12:39 speed 2.2x is ok, no practice just watch and see what's going on
____

TODO: note: Correct this notes as a practicie for learn english from <eng.to.correct>
---
title: "Challenges faced when researching RLHF with OpenAssistant"
seoDescription: "OpenAssistant addresses RLHF research: filling knowledge gaps, locating LLM cloud providers, updating documentation, and managing dataset preparation"
datePublished: Fri Aug 11 2023 03:47:57 GMT+0000 (Coordinated Universal Time)
cuid: cll61semr00030amf397xadu7
slug: challenges-faced-when-researching-rlhf-with-openassistant
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/Oaqk7qqNh_c/upload/5eb8c47dd3dbdd34054c37648e61e440.jpeg
tags: ai, data-science, data-science-training, llm, rlhf

---

At Dwarves, we've been working on researching various topics, focused on full-stack engineering as well as AI. One of my research goals was to find out how LLMs and RLHF training worked end-to-end through a chatbot interface:

%[https://www.youtube.com/watch?v=9wNsV3TTo-I] 

When working with LLMs and researching how to do RLHF training on OpenAssistant, I've faced quite a few challenges when handling operational, data, and foundational knowledge. About myself, I come from a background as a Frontend Engineer, so when starting to research LLMs, I encountered a handful of difficulties.

### **Lack of foundational knowledge**

It's necessary to have some sort of foundational knowledge before engaging in research, especially with AI and natural languages. Some concepts that I found were needed:

* **Epoch**: An epoch is a complete pass through the entire training dataset. During an epoch, each sample in the dataset has an opportunity to update the internal model parameters. If the entire dataset cannot be passed into the algorithm at once, it must be divided into mini-batches. The number of epochs is a hyperparameter that defines the number of times that the learning algorithm will work through the entire training dataset.
    
* **Batch size**: Batch size is the total number of training samples present in a single mini-batch. The entire dataset is divided into smaller parts called batches, which are passed through the model for training. The batch size and iterations are dependent on each other, so it is important to decide on these two parameters simultaneously to optimize the model.
    
* **Iteration**: An iteration is a single gradient update (update of the model's weights) during training. The number of iterations is equivalent to the number of batches needed to complete one epoch. For example, if the dataset consists of 1000 images and is divided into ten batches of 100 images each, then one epoch will involve ten iterations.
    

Additionally, you'll need to familiarize yourself with frameworks like PyTorch and have a basic understanding of concepts related to CUDA. You can find resources online to help bring in those concepts such as through YouTube tutorials like:

* [Standford's YouTube playlist](https://www.youtube.com/playlist?list=PLoROMvodv4rMiGQp3WXShtMGgzqpfVfbU)
    
* [MIT's YouTube playlist](https://www.youtube.com/playlist?list=PLoROMvodv4rMiGQp3WXShtMGgzqpfVfbU)
    
* [DeepLearning.AI](http://DeepLearning.AI)['s YouTube playlist](https://www.youtube.com/playlist?list=PLoROMvodv4rMiGQp3WXShtMGgzqpfVfbU)
    
* [Lex Fridman's up-to-date deep learning course](https://www.youtube.com/watch?v=0VH1Lim8gL8&list=PLrAXtmErZgOeiKm4sgNOknGvNjby9efdf&ab_channel=LexFridman)
    

### **Hard to find an LLM cloud provider**

Finding a cloud provider for LLM (Large Language Models) can be challenging due to the specialized nature of these applications. The nature of the space is still in its early stages and most cloud providers that are generally available today are specialized for enterprise use, rather than personal use. There are also search optimization issues, where just searching "LLM cloud providers" doesn't give much meaningful results.

There were a few providers that could have fit the bill:

* [Redmond AI](https://redmond.ai/) - offers accelerated cloud computing for cutting-edge AI and ML with affordable pricing.
    
* [Paperspace](https://www.paperspace.com/) - a cloud computing platform that offers high-performance computing resources, including GPU and CPU instances, as well as storage options similar to Google Colab
    
* [Skypilot](https://skypilot.readthedocs.io/en/latest/) - a framework for running LLMs, operated as a CLI, to manage executions for AI and batch jobs on any cloud such as Lambda, Google Cloud, Azure, etc.
    

Out of these choices, I chose Redmond AI as my cloud provider. They offer the necessary infrastructure to operate and train LLMs at a reasonable price with generally available documentation and support from their team.

### **Documentation has become obsolete**

The documentation for Open Assistant is comprehensive and easy to understand. However, since its platform has experienced rapid growth, some of the [documentation](https://projects.laion.ai/Open-Assistant/docs/intro) has become outdated. This is understandable given the challenge of maintaining up-to-date information and is often the challenge given to most open-source projects that have [rapid scale or development](https://github.com/LAION-AI/Open-Assistant/commits/main).

Fortunately, there is a vibrant [Discord server for the Open Assistant community](https://ykilcher.com/open-assistant-discord), which has been a valuable resource for me as a front-end engineer. I can ask questions and receive prompt assistance from active community members. While there are other channels for discussion, such as GitHub Discussions, they do not seem to be as active as those in the Discord server.

### **Dataset**

Preparing a dataset for research on Large Language Models (LLMs) can be a time-consuming process. Handling datasets is mostly a Data Analyst area of expertise, requiring cleaning data, making sure data is sufficiently de-aggregated, and having the dataset standardized for our particular set of training. We took a few examples from OpenAssistant's dataset as a reference to creating our own:

* [https://huggingface.co/datasets/OpenAssistant/oasst1](https://huggingface.co/datasets/OpenAssistant/oasst1)
    

In addition, collecting a lot of data for the dataset has also been a surprising challenge. Luckily, it shouldn't be too much of an issue regarding our research in the process of how we work with LLMs and how we operate with them. Below is the very benign dataset that we used to test our RLHF-trained model.

* [https://huggingface.co/datasets/toanbku/oa-df](https://huggingface.co/datasets/toanbku/oa-df)
    

### **Patience in the Training Process**

Training a machine learning model is a time-consuming process, unlike coding a web application. The training process can take **1-2 hours minimum**, also dependent on the size of the dataset and the complexity of the model. Only after the training process is complete can you check the results. This process is generally frustrating as almost all platforms will bill the training per hour and we can only know the result after everything is processed and done.

![Patience is a (value-added) virtue - IPEM](https://www.ipem-market.com/wp-content/uploads/Meme-No-rush.-Im-a-patient-bear..jpg align="left")

### Closing Thoughts

Researching RLHF training for LLMs has been an interesting journey and has helped me gain a lot more experience and knowledge of the platform than I would have just read sources online. There has been a great deal of support from the OpenAssistant discord community as well as from Redmond that has helped me in debugging various nuances in training and interacting with these models. A lot of unexpected challenges like a lack of foundational knowledge, difficulty finding an LLM cloud provider, dealing with obsolete documentation, preparing datasets, and requiring patience during the training process are very valuable experiences that I will take with me for, hopefully, a future project with AI.

### Contributing

At Dwarves, we encourage our people to read, write, share what we learn with others, and [contributing to the Brainery](https://brain.d.foundation/CONTRIBUTING) is an important part of our learning culture. For visitors, you are welcome to read them, contribute to them, and suggest additions. We maintain a monthly pool of $1500 to reward contributors who support our journey of lifelong growth in knowledge and network.

### Love what we are doing?

* Check out our [products](https://superbits.co/)
    
* Hire us to [build your software](https://d.foundation/)
    
* Join us, [we are also hiring](https://github.com/dwarvesf/WeAreHiring)
    
* Visit our [Discord Learning Site](https://discord.gg/dzNBpNTVEZ)
    

### Citations:

* [**https://towardsdatascience.com/epoch-vs-iterations-vs-batch-size-4dfb9c7ce9c9**](https://towardsdatascience.com/epoch-vs-iterations-vs-batch-size-4dfb9c7ce9c9)
    
* [**https://machine-learning.paperspace.com/wiki/epoch**](https://machine-learning.paperspace.com/wiki/epoch)
    
* [**https://machinelearningmastery.com/difference-between-a-batch-and-an-epoch/**](https://machinelearningmastery.com/difference-between-a-batch-and-an-epoch/)
    
* [**https://pub.towardsai.net/mastering-the-fundamentals-differences-between-sample-batch-iteration-and-epoch-bd5a33b2c5fd**](https://pub.towardsai.net/mastering-the-fundamentals-differences-between-sample-batch-iteration-and-epoch-bd5a33b2c5fd)
    
* [**https://www.sabrepc.com/blog/Deep-Learning-and-AI/Epochs-Batch-Size-Iterations**](https://www.sabrepc.com/blog/Deep-Learning-and-AI/Epochs-Batch-Size-Iterations)
    
* [**https://www.educative.io/answers/what-is-the-difference-between-epoch-batch-size-and-iteration**](https://www.educative.io/answers/what-is-the-difference-between-epoch-batch-size-and-iteration)
    
* [https://redmond.ai/](https://redmond.ai/)
    
* [https://www.paperspace.com/](https://www.paperspace.com/)
    
* [https://skypilot.readthedocs.io/en/latest/](https://skypilot.readthedocs.io/en/latest/)
    
* [https://huggingface.co/datasets/OpenAssistant/oasst1](https://huggingface.co/datasets/OpenAssistant/oasst1)
    
* [https://ykilcher.com/open-assistant-discord](https://ykilcher.com/open-assistant-discord)
    
* [https://projects.laion.ai/Open-Assistant/docs/intro](https://projects.laion.ai/Open-Assistant/docs/intro)
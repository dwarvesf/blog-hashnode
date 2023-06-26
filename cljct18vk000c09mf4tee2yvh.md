---
title: "Lessons Learned Building an LLM Chatbot: A Case Study"
datePublished: Mon Jun 26 2023 11:57:51 GMT+0000 (Coordinated Universal Time)
cuid: cljct18vk000c09mf4tee2yvh
slug: lessons-learned-building-an-llm-chatbot-a-case-study
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/eXmrW9I1Fnw/upload/4e0601766b2d640266b02d97c9043e9b.png
tags: challenge, ecommerce, chatbot, llm, chatgpt

---

When it comes to the ever-growing hype of GPT-based chatbots, we at [Dwarves Foundation](https://d.foundation/) felt it was time to dive in headfirst. Our objective was to build a lite version of a chatbot for an e-commerce partner. This chatbot aimed to consult and engage customers, nudging them towards purchasing products while later gathering critical data for further analysis and upselling.

## **The Story**

Chatbots have emerged as a powerful tool for customer engagement. Our challenge was to leverage the capabilities of GPT, the latest language model from OpenAI, to provide personalized product suggestions and detailed product information in a conversational manner.

## **The Implementation**

We integrated ChatGPT into the flow and applied the concept of prompt engineering to drive our bot's interactions. Every time a customer visited a product, we displayed a chatbox to provide support.

To tailor the interaction, we incorporated the context of the product — its name, category, price, description, rating, etc. — into a prompt for ChatGPT. We started by asking GPT to generate three popular questions to create a friendly welcome. From there, we answered questions based on the context we fed GPT, saving customers the effort of going through lengthy product descriptions.

![Basic flow to get the answer](https://cdn.hashnode.com/res/hashnode/image/upload/v1687779838327/8e5154e3-f0cb-4605-9e9a-247696086f0d.png align="center")

## **The Results**

Our venture into leveraging the MVP chatbot has shown some remarkable results. The chatbot was designed to answer basic product-related questions, and it met our expectations by providing correct responses 90% of the time. Questions such as “*where is this product made?” or* *“how long is the warranty?”* were handled effectively by the chatbot.

![Chatbot integration in the product detail page](https://cdn.hashnode.com/res/hashnode/image/upload/v1687779856702/6f8784e1-1cbb-449e-86c4-0e5b56a79d13.gif align="center")

What was surprising and exciting was its capability to deliver in-depth, encyclopedic knowledge about the products. In some instances, it served as an informative hub, answering inquiries such as *“What is Kali Citrate? Does it cause allergies?”* These instances showed the chatbot's potential to provide value beyond our initial expectations, enriching customer interaction by serving as a detailed information source.

However, this venture wasn't without its share of challenges. While the MVP exhibited immense potential, we also identified some issues that needed to be addressed for a successful full-scale production deployment.

## **The Challenges**

### **Consistency**

Language models like GPT can exhibit an idiosyncrasy that can pose challenges for real-world applications: even with the same input, the outputs can sometimes vary. This inconsistency is a particularly notable issue when dealing with questions that have different wording but similar meaning. Ensuring the chatbot consistently interprets and responds to these types of questions is a critical aspect of refining our implementation.

### **Accuracy**

The stakes are high when it comes to accuracy in AI applications, and this has proven to be a significant hurdle preventing many businesses from adopting Large Language Models (LLMs) like GPT. A single piece of misinformation could have far-reaching consequences, especially in items related to health and medicine. If a chatbot misinterprets a question or provides incorrect information about a product, it could potentially lead to reputational damage and legal liabilities.

Moreover, the chatbot needs to handle a wide array of questions from users and respond appropriately. It's not just about the capability to answer, but also the wisdom to understand when not to answer. Some questions might trigger responses that touch on politically sensitive issues or provide potentially harmful information, creating risks for the business.

### **Context Length**

The MVP chatbot was designed to incorporate product data into its responses, with the context forming the core of its understanding. However, GPT has a constraint of token limits within prompts, limiting the amount of text we could feed into the model. For some products and potential new use cases, a more extensive context is required to ensure accurate responses. This limitation doesn't just restrict the chatbot's effectiveness; it also raises cost considerations, as more extensive prompts would entail higher costs for using services like GPT. Furthermore, this problem extends to maintaining the chatbot's long-term memory. If a user's question relates to previous responses, the token limit makes it impossible to keep all historical threads within a single prompt, which can affect the continuity and relevance of the chatbot's responses.

### **Latency**

The time it takes for the AI service to respond is another critical challenge that we faced. Variability in latency while using an AI service like GPT can significantly affect user experience. A fast, seamless interaction is key to maintaining user engagement, and any delay in responses could result in a negative experience, thereby undermining the value of the chatbot. Developing solutions that can ensure consistent and swift responses is an integral part of our journey forward.

## **Conclusion**

Although we encountered several challenges in our MVP, these obstacles present valuable learning opportunities. Our R&D team is exploring various engineering practices to overcome these challenges and optimize our chatbot's capabilities. This exploration is not only an integral part of our development process but also a fascinating journey into the future of AI-powered customer service.
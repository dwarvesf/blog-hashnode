---
title: "Exploring Resumable Server-Side Rendering with Qwik"
seoTitle: "Exploring Resumable Server-Side Rendering with Qwik"
seoDescription: "Discover Qwik's resumable server-side rendering for fast web apps, reducing JavaScript and boosting performance via Dynamic Tree Shaking"
datePublished: Tue Aug 22 2023 07:48:19 GMT+0000 (Coordinated Universal Time)
cuid: cllm07wj4000l0al97w9agxuq
slug: exploring-resumable-server-side-rendering-with-qwik
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1692093137487/9cb21233-6cab-4386-95a1-661963c031dc.webp
tags: framework, javascript, ssr, server-side-rendering, qwik

---

Qwik is a front-end framework for creating web applications that offers lightning-fast page load times, regardless of the size and complexity of your site.

## **What is the problem Qwik is trying to solve?**

In collaboration with React, Next.js offers a solution by executing the **initial rendering on the server**, a practice known as **Server Side Rendering (SSR)**. This approach grants clients access to the interface and content prior to code download, benefiting user experience, and facilitating search engine bot indexing.

Nevertheless, after receiving the HTML, the browser proceeds to download JavaScript from the server and initiates the "Hydration" phase. During hydration, the browser takes on rendering responsibilities by synchronizing itself with the server. However, this implies that the same rendering performed by the server is duplicated by the browser before interactivity. The rendering process occurs twice: once on the server and once for the client to catch up.

In contrast, Qwik adopts a distinct approach instead of downloading the framework and repeating the server's rendering process. Qwik generates output with minimal JavaScript content and avoids hydration altogether, leading to remarkable performance speed by using Dynamic Tree Shaking and Resumable Rendering.

### Dynamic Tree Shaking

Dynamic tree shaking involves a concept within module bundling wherein the bundler can eliminate unused exports from the codebase, even in scenarios involving dynamic imports. This approach proves instrumental in minimizing the size of the final bundle, thus enhancing the speed of application loading.

Qwik's design is inherently compatible with the principles of tree shaking. Qwik's components undergo automatic segmentation into lazy-loaded segments by the [Optimizer](https://qwik.builder.io/docs/advanced/optimizer/index.mdx). Consequently, only the essential sections of your application are fetched as required, thereby diminishing the initial loading time.

### Resumability

Resumability is a key concept in Qwik applications that differentiates it from other frameworks. It refers to the ability of Qwik applications to resume from a server-side-rendered (SSR) state without the need for hydration.

In traditional SSR/SSG applications, when the application boots up on a client, it needs to restore three pieces of information:

1. Listeners - locate event listeners and install them on the DOM nodes to make the application interactive.
    
2. Component tree - build up an internal data structure representing the application component tree.
    
3. Application state - restore the application state.
    

This process is known as hydration. Hydration can be expensive because it requires downloading all of the component code associated with the current page and executing the templates associated with the components on the page to rebuild the listener location and the internal component tree.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692090153024/9287ba29-271f-4fd8-b4fa-956f1a78d721.png align="center")

Qwik is a **server-side framework**. In a sense, Qwik is **closer to Next JS than to React,** Qwik does a render of the HTML output using the JavaScript code. On the other hand, Qwik does not require hydration to resume an application on the client. This is what makes the Qwik application startup instantaneous. Instead of replaying all the application logic in the client (like in hydration), Qwik pauses execution in the server and resumes execution in the client.

Resumability is about pausing execution in the server and resuming execution in the client without having to replay and download all of the application logic.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692090165764/fbeede28-0932-40e0-9186-ffe6ea1bb38e.png align="center")

Completing the "**resumability**" puzzle is a compact Qwik Loader. This loader capitalizes on web workers to fetch the modest JavaScript segments generated during the initial compiler phase – the minuscule components essential for enabling page interactivity.

Here's an example:

Once the page is loaded, the page becomes interactive as quickly as possible. But no JS code related to the interaction is loaded.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692089980435/fce484d8-fb04-44f9-afa0-ffd4e62d1fcc.png align="center")

When click on the action button, a script for the interaction is downloaded from the server.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692090061993/9be49745-d706-4b22-968b-70a8ebb86790.png align="center")

## Tasks and Lifecycle

### Tasks

Tasks serve the purpose of executing asynchronous operations within component initialization or when there's a change in component state.

Tasks share similarities with React's `useEffect()`. Tasks exhibit similarities. However, key distinctions set Tasks apart from `useEffect`:

1. Tasks operate asynchronously.
    
2. Tasks execute both on the server and in the browser.
    
3. Tasks occur before rendering and have the potential to obstruct rendering.
    

### Lifecycle

The lifecycle of a component spans both server and browser runtimes. There are instances where the component is initially rendered on the server, while at other times, it might undergo rendering in the browser. Regardless of the scenario, the lifecycle sequence remains consistent; only the execution environment differs, either on the server or in the browser.

**Within the Qwik framework, the lifecycle of components is streamlined into 3 stages:**

* `Task` - Executes prior to rendering and is re-triggered upon changes in tracked state. (Tasks execute sequentially and block rendering.)
    
* `Render` - runs after the `Task` stage and before `VisibleTask`
    
* `VisibleTask` - runs after `Render` and when the component becomes visible
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692091913682/3b5517f1-70f8-4db2-929d-aed30f944c86.png align="center")

**SERVER**: Usually **the life of a component starts on the server** (during SSR or SSG), in that case, the `useTask$` and `RENDER` will run on the server, and then the `VisibleTask` will run in the browser, after the component is visible.

**BROWSER**: Sometimes a component will be first mounted/rendered in the browser, for example when the user SPA navigates to a new page, or a "modal" component first appears on the page. In that case, the lifecycle will run like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692092269721/9ed5bcab-be93-442b-809b-0ddc61c3489b.png align="center")

## **Is Qwik worth learning?**

The developer world is flooded with a plethora of frameworks vying for attention. Does it make sense to dive into another one?

Here's a twofold response to this.

Firstly, NextJS boasts a comprehensive ecosystem. It’s the reference implementation of React. NextJS has plenty of libraries that facilitate everything we easier aspire to achieve. It's complemented by in-depth documentation and a myriad of resources – aspects currently not paralleled by Qwik. Given these factors, it is skeptical about employing Qwik for critical projects, at least for the foreseeable future.

Yet, this merits emphasis, I'm convinced that Qwik's methodology represents the future of development. This makes it imperative to grasp the nuances of Qwik – its architecture, the secrets behind its efficiency, and the implications on our coding techniques, especially when dissecting code into minute fragments.

But it's not necessarily about all of us adopting Qwik down the line. Rather, the allure lies in the potential influence of Qwik's innovations on stalwarts like React and NextJS. Familiarizing oneself with Qwik can provide a pioneering advantage and a fresh lens through which to understand the dynamics of frameworks.

## Conclusion

With **resumability** It tries to fix the “issues” with **hydration**, for example. Nevertheless, the lazy loading of the code at the point when the user request it has some drawbacks too. This approach can lead to delays in downloading the necessary JavaScript and subsequent execution, thereby potentially diminishing the responsiveness of interactions, such as button clicks or intricate components with extensive functionality.

One major advantage of adopting Qwik is its swift initial page loading time. Particularly, if prioritizing Time To Interactive is paramount (resulting in enhanced SEO rankings on Google), Qwik presents itself as the optimal solution. Moreover, Qwik employs the familiar JSX/TSX syntax, akin to other prevailing frameworks.

### ***Contributing***

*At Dwarves, we encourage our people to read, write, share what we learn with others, and* [***contributing to the Brainery***](https://brain.d.foundation/CONTRIBUTING) *is an important part of our learning culture. For visitors, you are welcome to read them, contribute to them, and suggest additions. We maintain a monthly pool of $1500 to reward contributors who support our journey of lifelong growth in knowledge and network.*

### *Love what we are doing?*

* *Check out our* [***products***](https://superbits.co/)
    
* *Hire us to* [***build your software***](https://d.foundation/)
    
* *Join us,* [***we are also hiring***](https://github.com/dwarvesf/WeAreHiring)
    
* *Visit our* [***Discord Learning Site***](https://discord.gg/dzNBpNTVEZ)
    

## References

[https://www.kodaps.dev/en/blog/qwik-the-future-of-javascript-frameworks](https://www.kodaps.dev/en/blog/qwik-the-future-of-javascript-frameworks)  
[https://qwik.builder.io/docs/](https://qwik.builder.io/docs/)  
[https://www.adservio.fr/post/a-brief-history-of-web-apps-why-qwik-is-innovative](https://www.adservio.fr/post/a-brief-history-of-web-apps-why-qwik-is-innovative)
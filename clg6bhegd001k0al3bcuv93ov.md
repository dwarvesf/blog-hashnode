---
title: "What is PNPM?"
seoTitle: "What is PNPM?"
seoDescription: "PNPM is a package manager for Node.js which stands for “Performant NPM”. It was introduced in 2016, the same year Yarn was released. PNPM is a fast..."
datePublished: Fri Apr 07 2023 09:00:48 GMT+0000 (Coordinated Universal Time)
cuid: clg6bhegd001k0al3bcuv93ov
slug: what-is-pnpm
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680857748585/580ece0a-ca10-478e-ae97-f1a78f334bd4.png
tags: javascript, nodejs, package-manager, pnpm, nodemodules

---

*At Dwarves Foundation, we are always on the lookout for new tech. Researching PNPM was originally from research on what package manager* [*Next.js*](https://github.com/vercel/next.js/) *uses. We then tried to experiment with it for* [*dwarves/react-toolkit*](https://github.com/dwarvesf/react-toolkit/pull/46)*, which has given us some insights into some of the cost-benefits of using the package manager.*

## Introduction

PNPM is a package manager for Node.js which stands for “Performant NPM”. It was introduced in 2016, the same year Yarn was released. PNPM is a fast, disk space efficient package manager that supports monorepos. It creates a non-flat `node_modules` by default, so code has no access to arbitrary packages. PNPM performs installation in three stages:

1. **Dependency resolution** - The package manager identifies and fetches all required dependencies to the store.
    
2. **Directory structure calculation** - Based on these dependencies, it calculates the layout of the `node_modules` directory.
    
3. **Linking dependencies** - it retrieves and establishes hard links from the store to `node_modules` for all remaining dependencies.
    

## Advantages of PNPM

PNPM is a very performant alternative to most package managers. Here are a few advantages of using PNPM:

* Saves disk space by using a content-addressable store for packages
    
* Boosts installation speed with a three-stage process
    
* Creates a non-flat node\_modules directory for larger projects
    

### Saving Disk Space

Supposing you have 10 Node.js projects on your personal computer if you use NPM/Yarn, you will have 10 `node_modules` folders with a heavy size.

If you use PNPM, things will be different. It introduces a new concept to us, called a *content-addressable store*. See the image below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680857682673/a6fa2be4-8b7f-4c95-b650-857cb57782cb.png align="center")

*(source:* [*https://pnpm.io/motivation#saving-disk-space*](https://pnpm.io/motivation#saving-disk-space)*)*

As you can see, PNPM does not store packages in the `node_modules` folder, but rather in the content-addressable store. Therefore, in the `node_modules` folders of projects using PNPM, the packages are *linked* from the global store.

Thanks to this, package versions are only stored once on the disk

### **Boosting installation speed**

PNPM performs installation in three stages:

1. Dependency resolution: identifying and obtaining all necessary dependencies for the store.
    
2. Directory structure calculation: determining the layout of the `node_modules` directory based on these dependencies.
    
3. Linking dependencies: retrieving and establishing hard links from the store to `node_modules` for all remaining dependencies.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680857690990/42b1ade1-5769-4b98-b092-ed2eac9cb161.png align="center")

*(source:* [*https://pnpm.io/motivation#boosting-installation-speed*](https://pnpm.io/motivation#boosting-installation-speed)*)*

This approach is significantly faster than the conventional method of identifying, obtaining, and saving all dependencies directly to the `node_modules` directory.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680857706647/99dcc903-7618-4691-bda2-0812f2f7efe9.png align="center")

*(source:* [*https://pnpm.io/motivation#boosting-installation-speed*](https://pnpm.io/motivation#boosting-installation-speed)*)*

### **Creating a non-flat node\_modules directory**

First of all, we must ask why NPM chooses the flat `node_modules` structure approach.

Going back in time, before the release of NPM version 3, at this point, `node_modules` in NPM were still in a non-flat structure. As shown in the example below:

```jsx
node_modules
└─ foo
   ├─ index.js
   ├─ package.json
   └─ node_modules
      └─ bar
         ├─ index.js
         └─ package.json
```

This approach has some issues such as:

* The issue of long directory paths on the Windows operating system occurred because the package created a dependency tree that was too deep
    
* Packages were copy-pasted in many places because they were required in different dependencies
    

Therefore, to solve this problem, NPM decided to flatten `node_modules`. After NPM version 3, the structure of the `node_modules` directory will become like this:

```jsx
node_modules
├─ foo
|  ├─ index.js
|  └─ package.json
└─ bar
   ├─ index.js
   └─ package.json
```

Consequently, the source code can access dependencies that are not explicitly declared in the project. Following the example above, even though the project only uses the **foo** package, we can completely use the **bar** package without declaring it in package.json.

Unlike NPM version 3, PNPM tries to solve the issues without flattening the dependency tree. Follow the example below:

```jsx
node_modules
├─ foo -> .registry.npmjs.org/foo/1.0.0/node_modules/foo
└─ .registry.npmjs.org
   ├─ foo/1.0.0/node_modules
   |  ├─ bar -> ../../bar/2.0.0/node_modules/bar
   |  └─ foo
   |     ├─ index.js
   |     └─ package.json
   └─ bar/2.0.0/node_modules
      └─ bar
         ├─ index.js
         └─ package.json
---------------------------------------
->: a symlink (or junction on Windows)
```

The **foo** package still contains its dependency **bar** in the form of a symlink. And what’s special is that **foo** doesn’t have `node_modules` inside, this way the dependency tree of **foo** won’t be as deep as in NPM before the release of v3.

At first glance, the structure may seem complicated, but when working on larger projects you will see that this structure is clearer than NPM/Yarn

## Disadvantages of PNPM

However, there are a few disadvantages of using PNPM. One of them is that it can be slower than other package managers like Yarn or NPM when installing packages for the [first time](https://medium.com/@buffet_time/why-you-should-move-to-pnpm-82962f332418). It can also be difficult to use with some build tools like [Webpack](https://dev.to/stackblitz/what-is-pnpm-and-is-it-really-so-fast-and-space-efficient-29la).

PNPM's node\_modules layout uses [symbolic links to create a nested structure of dependencies](https://pnpm.io/symlinked-node-modules-structure). This has some implications for certain setups, where Windows machines or certain permissioned Linux environments may have trouble accessing these links.

Another potential issue with PNPM is that its nested dependency structure may not be compatible with certain older packages. [This can cause issues when trying to install packages that have these dependencies](https://pnpm.io/limitations).

## **Showcase**

Thankfully, PNPM is employed by numerous large companies, demonstrating its effectiveness. For an updated list, you can visit [https://pnpm.io/users](https://pnpm.io/users).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680857741782/5a265f11-55c1-4fba-938c-be260f1b2cb4.png align="center")

## Conclusion

PNPM is a package manager for Node.js that offers several advantages over other popular package managers, including saving disk space and boosting installation speed. It also creates a non-flat `node_modules` directory, which can be helpful for larger projects. However, there are some potential disadvantages to using PNPM, such as compatibility issues with certain older packages and slower initial package installation times. Despite these drawbacks, PNPM is used by numerous large companies and may be worth considering for your own projects.

Overall, PNPM offers some unique benefits and is a viable alternative to other package managers for Node.js. It's worth exploring whether it's the right choice for your specific use case.

## References:

* [https://pnpm.io/motivation](https://pnpm.io/motivation)
    
* [https://blog.bitsrc.io/pnpm-javascript-package-manager-4b5abd59dc9](https://blog.bitsrc.io/pnpm-javascript-package-manager-4b5abd59dc9)
    
* [https://pnpm.io/faq#what-does-pnpm-stand-for](https://pnpm.io/faq#what-does-pnpm-stand-for)
    
* [https://medium.com/pnpm/why-should-we-use-pnpm-75ca4bfe7d93](https://medium.com/pnpm/why-should-we-use-pnpm-75ca4bfe7d93)
    
* [https://medium.com/@buffet\_time/why-you-should-move-to-pnpm-82962f332418](https://medium.com/@buffet_time/why-you-should-move-to-pnpm-82962f332418)
    
* [https://dev.to/stackblitz/what-is-pnpm-and-is-it-really-so-fast-and-space-efficient-29la](https://dev.to/stackblitz/what-is-pnpm-and-is-it-really-so-fast-and-space-efficient-29la)
    
* [https://refine.dev/blog/pnpm-vs-npm-and-yarn/](https://refine.dev/blog/pnpm-vs-npm-and-yarn/)
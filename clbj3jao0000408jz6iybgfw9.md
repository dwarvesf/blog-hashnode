---
title: "Common challenges - Feature Flag"
seoTitle: "Feature Flag"
seoDescription: "A simple way to do this is create a product branch that goes from the current branch and develops in parallel into 2 different products"
datePublished: Sun Dec 11 2022 08:20:48 GMT+0000 (Coordinated Universal Time)
cuid: clbj3jao0000408jz6iybgfw9
slug: common-challenges-feature-flag
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1669606173262/GHMAvRPFG.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1669605317965/jhqxHng8i.png
tags: feature-flags, flag-arguments, feature-toggle

---

# Introduction:

* Currently, almost all modern software runs stably, but users often want to upgrade some functions or develop new features. After deciding to do a new function, the installation and testing process takes a long time to ensure that the new function works well and matches the needs of the user, without having to change too much. After developing the new feature, we will let some users use this function, while others will still use the current version of the application.
    
* In the process of developing new features, still ensuring the development and upgrading of old functions
    

![m518HUm.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669604815393/lzOabLtjf.png align="left")

A simple way to do this is to create a product branch that goes from the current branch and develops in parallel into 2 different products, but then managing and merging the code will be complicated and also very difficult. to manage as well as the cost to develop 2 versions at the same time. Thus we arise a requirement that allows the developer to change the operation of the application, the flow, the interface or the processing features without having to modify the code. To make it easy to imagine it as if you use Facebook with many features that it is only suitable for a certain number of users, or software versions with advanced features for paid users... The problem sounds complicated like how to determine which users are used, which functions are used or not, and how data is stored and returned to ensure security and not affect the data. Current function... However, we can divide it into many levels to implement this function such as installation at the application interface, installation at the API layer, installation at the database layer and deeper at the hardware level.

**So to store this information, where will we store and it, what solution:**

**Solution 1**: The flags are stored in the application's config, for example, config on a web server, web hosting, app config, and env in node js code ... this way will require the user to reload the application and will be difficult to manage

**Solution 2**: Self builds a feature flag system, build a flag admin portal, and the necessary services to operate a flag management page

**Solution 3**: Use the feature flag service from an external service.

## Solution

### Solution 1

For functions that are likely to change periodically or under certain conditions that you can manage or plan, you can save those settings in an environment variable. Today's source code platforms, all allow you to configure environment variables dynamically and allow you to relaunch or reload environment variables dynamically. In addition, this approach is suitable for small systems, the operator has good system administration ability. However, it is important to pay attention to security requirements when setting flags as well as re-testing

**Advantages:** Simple, easy to deploy with most platforms Fast deployment time Low deployment cost

**Defect:** Limited features, It only responds to turning on and off functions in a basic way Does not guarantee security requirements Implement guideline:

**Create config in env something like that:**

```bash
SHOW_REFERRAL_SCREEN=true
SHOW_MENU_SCREEN=true
SHOW_RESET_SCREEN=false
```

```javascript
function showUIRefferral() {
    if(env.SHOW_REFERRAL_SCREEN){
        // ... UI referral
    } else {
        // show UI empty
    }
}
```

### Solution 2

For medium to large projects with a good development team wishing to build feature flag functionality for long-term use with our products and services, the self-development option should be implemented. To develop a module like this the team will need to very clearly define the needs of the service, and analyze the fully functional design.

**Advantages:** Actively develop according to demand Easy to upgrade according to the design of the project Our system can scale and maintain easier Cost won’t increase so fast after our app has been scaled Developers can merge and release their new feature code without waiting too long to be merged, and user experiments won’t be affected after release ⇒ CI/CD

**Defect:** Development costs Functions are not rich Long development time Cost of personnel to operate the infrastructure system, automatically scaling

**Implement guideline -** **Approaches:** Implement authorization system with Module based for none authentication application Implement authorization system with Module based plus user role based if needed for authentication application

**Pros:** We can easily customize with more complex requirements later Easy to scale and decrease cost when compared to 3rd party

**Cons:** Takes time to initialize and implement in DevOps, Backend, Frontend, and Admin to manage. With a small team, it is often the case that we either can’t hit a deadline or have to accept tech debt later. What could go wrong? We can take too much time to implement our system and get results not as expected when lacking resources and time. **Design our own system Database** **Database name: Authorization List of tables**

![bZkNyXN.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669362920460/LZ6gvdsgC.png align="left")

#### Diagram for feature flag

##### System architecture

![HHb2c0t.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669362954457/Z3a_VivhD.png align="left")

**Backend** **Database migration** Follow the diagram above to write a migration API endpoint

* CRUD for Admin
    
    * Need to broadcast to socket channel on each action
        
* get-enabled-feature-flags endpoint to return list to FE App based on authentication
    

**Socket**

* Enable socket to help FE join to channel and listen topic
    
* Send socket event to FE after changing anything effect to feature flag
    

**Middleware** Input for middleware is a list of conditions with condition key and value, we will implement to get a list of feature flags that are enabled for these conditions

**\- Flow to get feature flags based on condition inputs**

![Pv1DJ3d-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669363115464/naxvmHR-O.png align="left")

**\- Flow to check condition is true or false in detail**

![j30A35J.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669363143664/F-nMYK9Dg.png align="left")

Example

Old men, children, and women OR man between 20 → 40 can see feature flag A

1. Create a feature flag: Name + Key
    
2. List and select existing triggers if not
    
3. Create the first trigger
    
    a. List variables
    

| Name | Variable Key | Variable Type |
| --- | --- | --- |
| Age | age | number |
| Sex | sex | enums |

b. List the condition

| Name | Variable Key | Condition Type | Value |
| --- | --- | --- | --- |
| Children | age | less\_than\_to\_equal | 16 |
| Old | age | greater\_than\_to\_equal | 60 |
| Woman | sex | equal | woman |

c. Create trigger: Name + Description + list conditions

1. Create the second trigger
    

a. List variables

| Name | Variable Key | Variable Type |
| --- | --- | --- |
| Age | age | number |
| Sex | sex | enums |

b. List condition

| Name | Variable Key | Condition Type | Value |
| --- | --- | --- | --- |
| Man Min | age | less\_than\_to\_equal | 40 |
| Man Max | age | greater\_than\_to\_equal | 20 |
| Woman | sex | equal | man |

c. Create trigger: Name + Description + list conditions

1. Add these triggers to the feature flag A
    

**Middleware**

1. Get information
    
2. Create list input conditions want to get feature flags: \[{key: “age”, value: 35}, {key: “sex”, value: “man}\]
    
3. SELECT \* FROM variables v WHERE v.key in \[”age”, “sex”\]
    
4. SELECT \* from conditions c WHERE c.variable\_id in \[variable\_ids\]
    
5. Get a list of true conditions
    
6. SELECT \* from triggers\_conditions tc WHERE tc.condition\_id in \[true\_condition\_ids\]
    
7. Loop triggers → filter trigger has all conditions in the trigger are a subset in true\_condition\_ids
    
8. SELECT \* from triggers\_flags tf WHERE tt.trigger\_id in \[enabled\_trigger\_ids\]
    
9. Return list of enabled unique feature flags
    

**Admin system**

For Homepage

![](https://i.imgur.com/Fr9pDDF.png align="left")

For create feature flag

![](https://i.imgur.com/L0IhgCv.png align="left")

For create trigger

![](https://i.imgur.com/Fr9pDDF.png align="left")

**Frontend Apps**

**Prerequisites**

* The backend and Frontend need to define a list of feature flag keys before. If we define before we can implement independency
    

**Implementation**

* FE Call to BE to get the list of available feature flags
    
* Save a list of available feature flags to context state or local storage
    
* Create util function isAllowFeature(’feature\_key’) → rendering condition
    

### **Solution 3**

For small and medium projects or MVP products that are in the process of testing, it is recommended to use support solutions from appropriate service providers to optimize development time and operating costs. as well as can quickly use many outstanding features compared to installing it yourself

* **Advantages**
    
    * Low service cost if few people are using the service
        
    * Many advanced functions
        
    * Rapid deployment
        
    * Multi-platform support with SDK
        
    * Real-time toggle feature
        
    * Support decentralization of management
        
* **Defect**
    
    * Service costs can be high when scale many users using the service
        
    * Unable to design the specific requirements of the project
        
    * Can’t solve some special requirement when improving feature
        

**Implementation guidelines:**

* Integrate SDK guideline: [https://docs.launchdarkly.com/home/getting-started/setting-up](https://docs.launchdarkly.com/home/getting-started/setting-up)
    
    This topic explains how to set up an SDK to begin using LaunchDarkly.
    
    **Setting up an SDK**
    
    The steps to integrate your application with LaunchDarkly are similar across all SDKs. We provide a variety of client-side, server-side, and mobile SDKs to choose from. To learn more about choosing an SDK, read [Client-side and server-side SDKs](https://docs.launchdarkly.com/sdk/concepts/client-side-server-side).
    
    We auto-generate documentation for most of our SDKs in GitHub Pages. The URL convention is `https://launchdarkly.github.io/<SDK-NAME>`. Some SDKs share functionality, which is reflected in their documentation. For example, our Node and JavaScript SDK have a common library found at [JS SDK Common](https://launchdarkly.github.io/js-sdk-common/).
    
* Setup code on the front end for each feature
    

```jsx
// To use in your project, import the initialize function
import { initialize } from '@devcycle/devcycle-js-sdk'

// The user object needs either a user_id, or isAnonymous set to true
const user = { user_id: 'my_user' }
let dvcClient

try {
  // Call initialize with the client key and a user object
  // await on the features to be loaded from our servers
  const dvcClient = await initialize('dvc_client_f895833e_2f80_4631_828f_e2fd5dc0f27b_23918a1', user)
                          .onClientInitialized()
  
  useDVCVariable()
} catch(ex) {
  console.log('Error initializing DVC: ${ex}')
}

function useDVCVariable() {
  if (!dvcClient) return
  
  // Fetch variable values using the identifier key coupled with a default value
  // The default value can be of type string, boolean, number, or object
  const dvcVariable = dvcClient.variable('liquidity-pool', false)
  if (dvcVariable.value) {
    // Put feature code here, or launch feature from here
  } else {
    // Put feature code here, or launch feature from here
  }
  
  // Register a callback to be notified when a value is updated
  dvcVariable.onUpdate((value) => {
    // updated variable value is available
  })
}
```

* Guide to Creating and manage Flags:
    
    * You can read the full guidelines [here](https://docs.launchdarkly.com/home/flags)
        
* A simple guide to creating a Flag
    
    * Step 1: Create a new feature flag
        

![](https://i.imgur.com/DSRtaFL.png align="left")

* Step 2: Define feature flag name
    

![](https://i.imgur.com/fbkXKRL.png align="left")

* Step 3: Add a condition to feature flag
    

![](https://i.imgur.com/YpRNvKR.png align="left")

|  | Solution 1: Setup config in the environment | Solution 2: Build a feature flag management system | Solution 3: 3rd party service |
| --- | --- | --- | --- |
| When implementing this solution | \- When the team can change env on the application easily so we can switch feature flags without rebuilding and touching to code - One source code and different feature flags status with multiple servers | \- Want to dynamic feature flags with multi variables - Feature flags rules and status changes regularly - The team has enough time and resources to implement it | \- Want to dynamic feature flags with multi variables - Feature flags rules and status changes regularly - Team willing to pay to save development time |
| Don’t require DevOps (difficult to deal with a big team) | ❌ | ☑️ | ☑️ |
| Other teams can switch feature flags | ❌ | ☑️ | ☑️ |
| No cost when scaling | ☑️ | ☑️ | ❌ |
| Save implement time and resource | ☑️ | ❌ | ☑️ |
| Custom dynamic and complex feature flag conditions | ❌ | ☑️ | ☑️ |
| Use case | \- Enable feature flag base on environments like sending login email on production but turning off on development and staging | \- Want to dynamic feature flags with multi variables |  |

### ***Contributing***

*At Dwarves, we encourage our people to read, write, share what we learn with others, and* [***contributing to the Brainery***](https://brain.d.foundation/CONTRIBUTING) *is an important part of our learning culture. For visitors, you are welcome to read them, contribute to them, and suggest additions. We maintain a monthly pool of $1500 to reward contributors who support our journey of lifelong growth in knowledge and network.*

### *Love what we are doing?*

* *Check out our* [***products***](https://superbits.co/)
    
* *Hire us to* [***build your software***](https://d.foundation/)
    
* *Join us,* [***we are also hiring***](https://github.com/dwarvesf/WeAreHiring)
    
* *Visit our* [***Discord Learning Site***](https://discord.gg/dzNBpNTVEZ)
    
* *Visit our* [*GitHub*](https://github.com/dwarvesf)
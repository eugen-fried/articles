# An Approach to Effective Multi-Environment Configuration Management pt.1
## Problem Description
The day has come. Your project now is mature enough to be deployed in production. Of course, you ran all your checks in the QA environment, rolled out everything to Staging to have a perfect copy of your production setup and 
now you want to go ahead and deploy your app backend servers in three regularlocations: North America, Europe and Asia. You even have Kubernetes setup ready, because why not? 
We should be ready in case of some proper load. And then you start to think about the obvious thing: How do you maintain your application configuration files between the environments?
## The Naive Approach (#1)
How did you managed it before? Well, your framework or build system has this beautifull concept of profiles. You had your *application-local.properties*, *application-dev.properties*, *application-qa.properties* and *application-dev.properties* 
files sitting in your static resources folder. Your framework or build system just needs to know which profile we are aiming at and it will pick up the appropriate config file itself. We can just add there all releveant environment files and we are good to go.
Well, no. It is production and we need to separate our configuration from code as there might be some sensitive stuff. Let's do a list of pros and cons for this approch:

### Pros
- It is under developers' fingertips. Any change is easy to do.
- The build system or framework support this approach out of the box without any need to customize the build or run process besides setting an environment variable for the profile.
### Cons
- As mentioned earlier, this approach is not secure. If the codebase is compromised, all production information is compromised as well.
- It is harder to do quick fixes, as the code and the configuration are in the same repository. The branching model you have for your code now affects your configuration as well.
- Developers do tend to play with the configs a lot. I know, you have proper code reviews, but human error can happen and wrong commit to a config file might go to prod.
Overall this approach is oversimplified and usually works only for pre-production stages of development.


## The Regular Approach (#2)
Let's now take the path many companies take. They have a separated repository for the configuration files for all services where they put the settings according to the environment. Only system engineers are allowed to commit to those repositories, so any change
will be reviewed at least twice. We can still keep the QA in the original repo, so the devs could ease the test environment configuration, but from staging and on everything goes according to production workflow. We can even use something like Amazon SSM to
obscure the sensitive information from people who have access to this repository.
### DRY and Diverged Configs
The biggest issue with having a specific file for each environment is maintaining those files in the long run. Things tend to change and copy-pasted configs start to diverge. For example, we want to change the resources used by our service and scaling settings 
for each environment. Another example - our Kafka Streams application might recuire some fine-tuning to handle the load in a specific geographic region. The more time passes from the initial deployment the more the files start to differ, yet maintaining 
similar structure and many values. It's becoming more and more difficult to introduce new features which makes the system once again human error prone.

### Pros
- Only system engineers can access the application configuration.
- Code and configuration are separated hence we have smaller chance of human error and branching model should not affect the production behaviour.
### Cons
- This approach requires some custom tinkering during the CI stage. Now you need to checkout two repositories and pick the appropriate file from another repository.
- Developers have harder time to introduce changes to the configuration.
- The biggest issue (which is shared with our Naive approach as well) - it's hard to maintain because of the code duplication.


## Can We Do Better? (Approach #3)
Let's take a look at a the very specific problem we want to solve. On one hand we have a lot of code that is shared among the "siblings", on the other there are parts that differ, some differ in every deployment while others are just specific exceptions.
Does this look familiar? Of course it does, it reminds us of one of the greatest principles of object oriented programming: hierarchy. A generic configuration which is never used on it's own containing all relevant placeholders and common configs is our 
abstract class, specific environments are subclasses and this also gives us the possibility to "extend subclasses" (having a customer-specific deployment for example). So now the question is, how could we implement the approach in place and make 
the configuration files act like OOP classes.
Note: I have to give credits for the idea my previous tech lead Grigory Tukmachev who insisted on this approach instead of simple copy pasting. Huge kudos to him, as it made the almost impossible task of configuring and debugging ~30 modules in 5 
production environments a little more manageable.
### Tools at Our Disposal
Being a mostly JVM programmer I will use the following two tools to cope with this task: Thymeleaf template engine and Typesafe config. I'm sure any other mature programming environment should have something similar, but from now on we are in the JVM world.

# An Approach to Effective Multi-Environment Configuration Management pt.1
## Problem Description
The day has come. Your project now is mature enough to be deployed in production. Of course, you ran all your checks in the QA environment, rolled out everything to Staging to have a perfect copy of your production setup and now you want to go ahead and deploy your app backend servers in three regularlocations: North America, Europe and Asia. You even have Kubernetes setup ready, because why not? We should be ready in case of some proper load. And then you start to think about the obvious thing: How do you maintain your application configuration files between the environments?
## The Naive Approach (#1)
How did you managed it before? Well, your framework or build system has this beautifull concept of profiles. You had your *application-local.properties*, *application-dev.properties*, *application-qa.properties* and *application-dev.properties* files sitting in your static resources folder. Your framework or build system just needs to know which profile we are aiming at and it will pick up the appropriate config file itself. We can just add there all releveant environment files and we are good to go.
Well, no. It is production and we need to separate our configuration from code as there might be some sensitive stuff. Let's do a list of pros and cons for this approch:
### Pros
- It is under developers' fingertips. Any change is easy to do.
- The build system or framework support this approach out of the box without any need to customize the build or run process besides setting an environment variable for the profile.
### Cons
- As mentioned earlier, this approach is not secure. If the codebase is compromised, all production information is compromised as well.
- It is harder to do quick fixes, as the code and the configuration are in the same repository. The branching model you have for your code now affects your configuration as well.
- Developers do tend to play with the configs a lot. I know, you have proper code reviews, but human error can happen and wrong commit to a config file might go to prod.
Overall this approach is oversimplified and usually works only for pre-production stages of development.

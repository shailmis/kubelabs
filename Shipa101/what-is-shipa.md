# Application release frameworks

Application release frameworks or application orchestration tools have flooded the market in recent years, growing popular by the day. You may know these tools more commonly as CI/CD tools such as Azure DevOps, Jenkins, etc... While the aforementioned pipelines are more general purpose and can be used to orchestrate releases in all sorts of environments, more specific tools exist for Kubernetes. This includes tools such as [ArgoCD](../GitOps101/argocd.md), which waits until changes to the Kubernetes resource yaml are merged into master, at which point ArgoCD intelligently updates the cluster with the new resource yaml.

Now, we would like to bring a slightly different tool that caters to a different need: [Shipa](https://learn.shipa.io).

## What does it do?

First of all, this is not a replacement for ArgoCD. In fact, many users regularly run both ArgoCD and Shipa together. A good example of this can be found on the [introductory page of Shipa](https://learn.shipa.io/docs). The problem Shipa solves is much different.

When a new developer joins your team, there is a possibility that they might not know much about Kubernetes. After all, there is no need for a regular front-end developer to know the internals of how your complex Kubernetes cluster works. So, while they might be excellent at their job as a front-end developer, they might be slowed down and have to spend time learning how Kubernetes works so that they can effectively build your application. This is a pretty big problem, since Kubernetes has a pretty complex way of working, and can be difficult to pick up while working on a large project. It would also not be enough to learn Kubernetes alone since your application might have a specific Kubernetes architecture that has to be learned as well. Once all that is done, and the developer does changes to a resource, there is an extensive testing process that must be followed. The developer should either run a local Minikube cluster and deploy the resources to that, or have something like a [Virtual cluster](../Loft101/what-is-loft.MD) that they can deploy to.

This is where Shipa steps in, and allows developers to easily run code on Kubernetes clusters without having to have knowledge of how the cluster works. That information is abstracted away and can be dealt with by a DevOps team that would have much more knowledge on how Kubernetes works. Compared to the alternative situation where the DevOps team may have to hunt down any issues caused in the cluster by the developers, this is far easier and gives much less overhead to the DevOps teams.

## How does it do it?

Shipa works by allowing the creation of frameworks within clusters. These frameworks allow the DevOps team to specify things such as the security rules, permission, etc... which will be imposed on the developers who will be using the clusters. So the developer only needs to write code and commit it, at which point a CI/CD pipeline would deploy the changes to the Shipa pool. This includes all the security checks, code quality checks, and other checks that run before the Kubernetes resource gets created.

So this means that the developer has to spend time developing, and no time working with the DevOps part of the application. On the other hand, the DevOps team has full control over the whole process, from the clusters to the configurations.

Now that you know what Shipa is, let's take a hands-on approach to work with Shipa in the Shipa lab.

[Next: Shipa lab](./shipa-lab.md)
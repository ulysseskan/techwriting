# Nixpacks vs. Buildpacks vs. Dockerfiles: Exploring Modern App Packaging


# **Summary**

This article explores the differences between modern app packaging tools: Dockerfiles, Buildpacks, and Nixpacks.



* Dockerfiles provide a simple language for creating images, but it is easy to create an insecure, space-wasting, or resource-intensive Dockerfile.
* Buildpacks provide many advantages over Dockerfiles, including reusability, modularity, and safety. However, they lack flexibility, especially for less common languages, and have longer build times due to dependency analysis.
* Nixpacks provide a more approachable and less cumbersome way to make reproducible builds, a standard for fully reproducible builds, more extensibility, and more understandable error messages. Nixpacks are also more transparent, allowing you to audit the plan it creates before building.


# **Dockerfiles**

Dockerfiles provide a simple language for describing images.  They’re good if you’re an operator or platform implementer.  You can create base images for your organization.  You can easily run your own tests on the image locally before uploading.  And they provide a level of transparency.

<p align="center">
  <img src="https://github.com/ulysseskan/techwriting/assets/71786368/ca25ffe4-2c00-4831-94ae-be561af8eae6" />
  <br>
  Building an image using a Dockerfile, as drawn by <a href="https://iximiuz.com/en/posts/you-need-containers-to-build-an-image/">Ivan Velichko</a>
</p>


Dockerfiles appear trivial and usually start trivial, but don't remain trivial for long. It’s easy to inadvertently write a Dockerfile that is insecure, wastes space<sup><a href="#bookmark=id.fytznfa6lti1">1</a></sup>, promotes build churn, or more  frequently: all three.


<p align="center">
  <img src="https://github.com/ulysseskan/techwriting/assets/71786368/85f40336-1f36-4f5d-8d8b-099eba854fa5" />
  <br>
  Two Dockerfiles - the one on the right is more efficient
</p>


<p align="center">
  <img src="https://github.com/ulysseskan/techwriting/assets/71786368/57bdbcc1-0f0b-4f65-9582-ad4caba04f24" />
  <br>
Building the app with the improved Dockerfile took 8.3 seconds (vs. 50.2 seconds for the original one, not shown)
</p>




# **Buildpacks**

All the tips and tricks that you need to know in order to write production grade Dockerfiles are given for free in a buildpack. You don't need to think about the efficient ordering and contents of layers, the buildpack does it. You don't need to worry about mystery dependencies arriving via `wget`, buildpacks control what dials out to the outside world. Build churn is dropped to the minimum necessary changes, rather than the minimum possible changes. Disk space and network traffic is saved because far more layers are identical.  Buildpacks provide a framework to ensure your fleet of containers use compliant images, plus an easy way to patch vulnerabilities without needing to involve the dev team.  


<p align="center">
  <img src="https://github.com/ulysseskan/techwriting/assets/71786368/d9d2d20b-ed1e-4ded-9293-0d62c00eb2ec" />
  <br>
How buildpacks work, from <a href="https://buildpacks.io/docs/concepts/](https://buildpacks.io/docs/concepts/">buildpacks.io</a>
</p>


## Advantages over Dockerfiles

Buildpacks have many advantages over Dockerfiles.  The main advantage is they sit outside your app, separating you as a developer from lower level infrastructure concerns.  Also, they’re:



* Reusable: You can use the same buildpack for many apps, instead of creating a new Dockerfile for every repo.  You don’t need a Dockerfile for each app.
* Sometimes Faster: Layers are only rebuilt/uploaded when necessary, saving disk space and traffic.  If you’re upgrading your JDK version for example, it will install the JDK, but if you’re upgrading your OS, it’ll keep the same JDK.  It’s app aware and knows when it needs to bust your cache and when it doesn’t.  Dockerfiles would require rebuilding the whole image when the OS changes.
* Modular: You can combine buildpacks to create composite images.  For example, if you have an app that uses both Java and Node runtimes, with a Dockerfile you’d have to do some custom work, but with buildpacks you just combine the JavaScript buildpack with the Java buildpack and they’ll run against your app.  (As a bonus, the Java buildpack will calculate optimal JDK memory flags based on available resources.)
* Safe: They allow you to meet image security requirements without dev intervention.  If you need to update OpenSSL for example, you don’t need to modify your code.  Buildpacks are popular in the finance security and compliance space for this reason.





## Disadvantages



* Buildpacks are mainly focused on popular languages and frameworks, like Node.js, Java, Go, Python, etc.  If you’re working with less common or niche languages, you may find limited or no pre-existing buildpacks available.  (though you could delve into creating your own, this will take quite a bit of effort diving into the spec)
* They don’t offer as much flexibility for customization compared to manual Dockerfile-based builds.  You may find it challenging to modify the behavior of existing buildpacks.
* Dependency analysis and resolution makes for longer build times.
* The build process is abstracted, and limited if you need to execute specific commands or scripts at specific stages.


# **Nixpacks**

Nixpacks emerged from Railway as an alternative to Buildpacks, specifically addressing the shortcomings experienced when deploying thousands of different user apps to the Railway platform.  From the [Nixpacks Github page](https://github.com/railwayapp/nixpacks):


    _Nixpacks takes a source directory and produces an OCI compliant image that can be deployed anywhere. … The biggest change [from Buildpacks] is that system and language dependencies are pulled from the Nix ecosystem._

Nixpacks make it trivial to build working OCI compliant images that can be run on any platform without needing to understand how Dockerfiles/Container Filesystem/BuildLayers work.  Familiarity with Docker and Dockerfile syntax is not universal.  Even those with experience may not be equipped to write a Dockerfile that can quickly build (and rebuild) a small and secure image.


## Advantages over Buildpacks



* Nixpacks make **reproducibility** more approachable/less cumbersome
* Create a **standard** for fully reproducible builds
* Are more easily **extensible** (--pkgs)
* Error messages are **more understandable**
* Can spit out a **[plan](https://nixpacks.com/docs/how-it-works#plan)**, which, when run again, will assure that build artifacts are exactly the same every time it's run against that plan.  The result is that any code built with Nixpacks has all its dependencies snapshotted so it won't break over time.  
    * This means that any time someone comes to Railway and deploys a template, it won’t be broken.

<p align="center">
  <img src="https://github.com/ulysseskan/techwriting/assets/71786368/416d04ad-38d0-459f-93d8-7947c16c232e" />
  <br>
How Nixpacks works: Detect language > Plan > Generate Dockerfile > Build, as drawn by <a href="https://www.qovery.com/blog/my-feedback-about-nixpacks-an-alternative-to-buildpacks">Romaric Philogène</a>
</p>

Nixpacks is both simpler and more transparent than buildpacks.  You can audit the plan it creates before you build:

<p align="center">
  <img src="https://github.com/ulysseskan/techwriting/assets/71786368/f266b63a-11f0-439d-ab3e-1bd5060f1190" />
  <br>
Auditing the nixpacks plan
</p>

<p align="center">
  <img src="https://github.com/ulysseskan/techwriting/assets/71786368/dafb321a-b101-4d9a-ab5e-d3f158fb02c8" />
  <br>
Building the same app with Nixpacks took 38.3 seconds (subsequent builds were shorter)
</p>



The biggest change with Nixpacks is that the system and language dependencies are pulled from the Nix ecosystem.  This is great when something goes wrong, as Nix error messages are generally more understandable vs. what you can find in buildpacks. (ref: [error 1](https://stackoverflow.com/questions/67537907/buildpack-unable-to-resolve-artifact-for-gradle-multiproject-java-springboot-cod), [error 2](https://github.com/GoogleCloudPlatform/buildpacks/issues/14))


## Disadvantages



* Nixpacks is new.  Community support is not as high.
* Alpine, a small and secure linux distro, can’t be used as a base image yet ([#729](https://github.com/railwayapp/nixpacks/issues/729)).  Manually written Dockerfiles have the advantage here.
* Final image sizes can be fairly large ([#139](https://github.com/railwayapp/nixpacks/discussions/139)).
* [File-based configuration](https://nixpacks.com/docs/configuration/file) is still in the experimental stage. ([#508](https://github.com/railwayapp/nixpacks/discussions/508))


# Quick Syntax Examples



* `Dockerfile: Build a docker image using a specific Dockerfile`


```
docker build --file Dockerfile .


```



* `Buildpacks: Decide on a builder, then build an app with the name sample-app`


```
pack builder suggest
pack build sample-app --path samples/apps/java-maven --builder cnbs/sample-builder:jammy


```



* `Nixpacks: Create an image with the name my-app`


```
nixpacks build ./path/to/app --name my-app
```



# Comparison Chart


<table>
  <tr>
   <td>
   </td>
   <td><strong>Dockerfiles</strong>
   </td>
   <td><strong>Buildpacks</strong>
   </td>
   <td><strong>Nixpacks</strong>
   </td>
  </tr>
  <tr>
   <td><strong>Audience</strong>
   </td>
   <td>Operators, Platform Implementers
   </td>
   <td>Developers
   </td>
   <td>Developers
   </td>
  </tr>
  <tr>
   <td><strong>Best for Building</strong>
   </td>
   <td>Base Images, OS Images, some Apps
   </td>
   <td>Apps
   </td>
   <td>Apps
   </td>
  </tr>
  <tr>
   <td><strong>Language support (out of the box)</strong>
   </td>
   <td>Infinite
   </td>
   <td>12 (officially)
   </td>
   <td><a href="https://nixpacks.com/docs/providers">20</a>
   </td>
  </tr>
  <tr>
   <td><strong>Ease of extending</strong>
   </td>
   <td>Easy, just add dependencies
   </td>
   <td>Hard.  Lengthy spec<sup><a href="#bookmark=id.ejvkpjsoauju">2</a></sup>, but can be done with buildpack.toml etc.
   </td>
   <td>Intermediate.  (<code>--pkgs</code> option) Requires some Nix packaging knowledge 
   </td>
  </tr>
  <tr>
   <td><strong>Auto-Detects Languages</strong>
   </td>
   <td>No
   </td>
   <td>Yes
   </td>
   <td>Yes
   </td>
  </tr>
  <tr>
   <td><strong>Ease of getting started</strong>
   </td>
   <td>Intermediate to Hard
   </td>
   <td>Intermediate
   </td>
   <td>Easy
   </td>
  </tr>
  <tr>
   <td><strong>License</strong>
   </td>
   <td>N/A
   </td>
   <td>Apache-2.0
   </td>
   <td>MIT
   </td>
  </tr>
  <tr>
   <td><strong>OCI compliant images (portable, verifiable)</strong>
   </td>
   <td>No, unless you follow the spec / use an OCI compliant exporter
   </td>
   <td>Yes
   </td>
   <td>Yes
   </td>
  </tr>
  <tr>
   <td><strong>Integrations</strong>
   </td>
   <td>Amazon ECS, Azure, CircleCI, GitLab, Google, Tekton, …
   </td>
   <td>Azure, CircleCI, GitLab, Google, Heroku, Spring Boot, Tekton, …
   </td>
   <td><a href="https://nixpacks.com/docs/deploying/railway">Railway</a>, Flightcontrol, Easypanel, … \

   </td>
  </tr>
</table>



# Overall

The choice between Dockerfiles, Buildpacks, and Nixpacks depends on the specific needs and requirements of your project.  But the ease of use of Nixpacks makes it a serious contender.

Nixpacks is an open source project developed by Railway.  Check out Railway today for easy, fast, and efficient cloud hosting of your apps.  [A free starter plan is available](https://railway.app/pricing).


# Alternatives

You can build images without Docker/Dockerfiles, using:



* [Nix with tar, sha256sum, and jq](https://news.ycombinator.com/item?id=32502622)
* [Bazel’s rules_docker](https://github.com/bazelbuild/rules_docker)
* [Packer and Ansible](https://alex.dzyoba.com/blog/packer-for-docker/)
* [Red Hat's buildah](https://www.redhat.com/sysadmin/building-buildah)


# Footnotes

<sup>1</sup> The [Dive tool](https://github.com/wagoodman/dive) can help you explore each image layer and find wasted space.

<sup>2</sup> Long [buildpack spec](https://github.com/buildpacks/spec/blob/main/buildpack.md) is long.


## References

Nixpacks takes a source directory and produces an OCI compliant image
[https://news.ycombinator.com/item?id=32501448](https://news.ycombinator.com/item?id=32501448)

My Feedback about Nixpacks - an alternative to Buildpacks by Romaric Philogène
[https://www.qovery.com/blog/my-feedback-about-nixpacks-an-alternative-to-buildpacks](https://www.qovery.com/blog/my-feedback-about-nixpacks-an-alternative-to-buildpacks)

Buildpacks vs. Dockerfiles by Genevieve L'Esperance
[https://technology.doximity.com/articles/buildpacks-vs-dockerfiles](https://technology.doximity.com/articles/buildpacks-vs-dockerfiles)

Defence Against the Docker Arts by Joe Kutner (Heroku)
[https://www.youtube.com/watch?v=ofH9_sE2qy0](https://www.youtube.com/watch?v=ofH9_sE2qy0)

How Docker Build Command Works Internally by Ivan Velichko
[https://iximiuz.com/en/posts/you-need-containers-to-build-an-image/](https://iximiuz.com/en/posts/you-need-containers-to-build-an-image/)


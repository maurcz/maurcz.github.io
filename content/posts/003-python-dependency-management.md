---
title: "Python Dependency Management"
date: 2021-07-12T20:29:25-04:00
categories: ["Python", "Technical"]
tags: ["dependency", "management", "pip", "poetry", "pipenv", "pip-compile", "lockfile", "safety", "dependabot", "tox"]
summary: A guide on the whys behind taming your python dependencies, with lots of recommendations.
---

Dependencies are not going anywhere anytime soon.

One of the major things that move the tech space forward is the ability to combine packages created by others when you're giving life to your own project. And as any developer that's been around for a while, I can tell you the list of dependencies in a system tend to grow **tremendously** over time - be it third-party or internal ones. How do you handle all of that?

Through suffering, pain and despair... or by learning proper dependency management.

Taming the dependency beasts is way harder than most developers would anticipate. 2 years ago, when I started working with Python, it shocked me when I realized that bare `pip` provided pretty minimal _proper_ dependency management support (but things have changed ever since - more on that later). The ecosystem has the likes of `poetry`, `pipenv` and `pip-compile` to fill in the gaps but, in my experience, there are people that still think the `pip` + `requirements.txt` combo is more than enough. Others, especially the ones that use python to achieve _something_ but have no software development background (like most Data Scientists I've met), really struggle to understand why we need all the extra controls around a simple `pip install`.

I've had my _fair_ share of dependency issues ever since I became a python developer, and my intention with this post is to share everything I've learned along the way. I'm also aiming to focus on the _whys_ behind every decision - something I feel is often lacking for dependency management explanations. The topics are supposed to build on top of each other, so you're free to skip the ones you've already mastered.

Welcome to the sea of versions, where a 0.1.0 that depends on a 4.2.1 could totally break your 2.3.0 if you don't install the new 3.4.1 patch for your dependency management tool.

## Table of Contents

- [Underlying Dependencies](#underlying-dependencies)
- [Dependency Resolution](#dependency-resolution)
- [Lockfiles](#lockfiles)
- [Package Integrity](#package-integrity)
- [Private Feeds](#private-feeds)
- [Vulnerability Scans](#vulnerability-scans)
- [Testing Against Multiple Python Versions](#testing-against-multiple-python-versions)
- [Review](#review)

## Underlying Dependencies

When you start working with Python, people tell you about the magic that `pip` provides: through a few simple commands, you get access to [lots](https://pypi.org/simple/) of interesting packages to use in your projects. However, in my experience, it takes some time for newcomers to realize that one package might depend _on **many** others_.

Let's use the popular package publisher `twine` as an example. All you have to do to install this package is `pip install twine`. Pip, by default, will hit [PyPi](https://pypi.org/project/twine/), download/install the package and also _all of twine's dependencies_. Here's the dependency graph for `twine`:

```python
twine==3.4.1
  - colorama [required: >=0.4.3, installed: 0.4.4]
  - importlib-metadata [required: >=3.6, installed: 4.5.0]
    - zipp [required: >=0.5, installed: 3.4.1]
  - keyring [required: >=15.1, installed: 23.0.1]
    - importlib-metadata [required: >=3.6, installed: 4.5.0]
      - zipp [required: >=0.5, installed: 3.4.1]
  - pkginfo [required: >=1.4.2, installed: 1.7.0]
  - readme-renderer [required: >=21.0, installed: 29.0]
    - bleach [required: >=2.1.0, installed: 3.3.0]
      - packaging [required: Any, installed: 20.9]
        - pyparsing [required: >=2.0.2, installed: 2.4.7]
      - six [required: >=1.9.0, installed: 1.16.0]
      - webencodings [required: Any, installed: 0.5.1]
    - docutils [required: >=0.13.1, installed: 0.17.1]
    - Pygments [required: >=2.5.1, installed: 2.9.0]
    - six [required: Any, installed: 1.16.0]
  - requests [required: >=2.20, installed: 2.25.1]
    - certifi [required: >=2017.4.17, installed: 2021.5.30]
    - chardet [required: >=3.0.2,<5, installed: 4.0.0]
    - idna [required: >=2.5,<3, installed: 2.10]
    - urllib3 [required: >=1.21.1,<1.27, installed: 1.26.6]
  - requests-toolbelt [required: >=0.8.0,!=0.9.0, installed: 0.9.1]
    - requests [required: >=2.0.1,<3.0.0, installed: 2.25.1]
      - certifi [required: >=2017.4.17, installed: 2021.5.30]
      - chardet [required: >=3.0.2,<5, installed: 4.0.0]
      - idna [required: >=2.5,<3, installed: 2.10]
      - urllib3 [required: >=1.21.1,<1.27, installed: 1.26.6]
  - rfc3986 [required: >=1.4.0, installed: 1.5.0]
  - tqdm [required: >=4.14, installed: 4.61.1]


# NOTE: dependency graph generated using a dependency management tool
# - "required" are directives specified by twine
# - "installed" is the version that the tool decided to install.
```

That's way more packages than someone would expect, right? Dependency chains can get super complicated, like `twine > readme-rendered > bleach > packaging > pyparsing`. Also notice how there are some _repeated_ entries for a few packages. `requests` is both required by `twine` directly and `requests-toolbelt`.

The key take-away here is realizing that this is the graph for _one single_ dependency. Your large project can easily go above the mark of 100 total packages, even though all you're asking for are 20. After all, if you can ask for dependencies, so can the creators of the dependencies you're using, right?

But how are those chains generated? Projects often have a file called `setup.py` which can be used to specify the _underlying dependencies_ that are required if someone ever tries to install the project as a package. Here's an excerpt of the `setup.py` [from](https://github.com/psf/requests/blob/v2.25.1/setup.py) the `requests` package:

```python
requires = [
    'chardet>=3.0.2,<5',
    'idna>=2.5,<3',
    'urllib3>=1.21.1,<1.27',
    'certifi>=2017.4.17'

]
test_requirements = [
    'pytest-httpbin==0.0.7',
    'pytest-cov',
    'pytest-mock',
    'pytest-xdist',
    'PySocks>=1.5.6, !=1.5.7',
    'pytest>=3'
]

(...)

setup(
    name=about['__title__'],
    version=about['__version__'],
    description=about['__description__'],
    long_description=readme,
    long_description_content_type='text/markdown',
    author=about['__author__'],
    author_email=about['__author_email__'],
    url=about['__url__'],
    packages=packages,
    package_data={'': ['LICENSE', 'NOTICE']},
    package_dir={'requests': 'requests'},
    include_package_data=True,
    python_requires=">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*",
    install_requires=requires,
    license=about['__license__'],
    zip_safe=False,
    classifiers=[
        'Development Status :: 5 - Production/Stable',
        'Intended Audience :: Developers',
        'Natural Language :: English',
        'License :: OSI Approved :: Apache Software License',
        'Programming Language :: Python',
        'Programming Language :: Python :: 2',
        'Programming Language :: Python :: 2.7',
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.5',
        'Programming Language :: Python :: 3.6',
        'Programming Language :: Python :: 3.7',
        'Programming Language :: Python :: 3.8',
        'Programming Language :: Python :: 3.9',
        'Programming Language :: Python :: Implementation :: CPython',
        'Programming Language :: Python :: Implementation :: PyPy'
    ],
    cmdclass={'test': PyTest},
    tests_require=test_requirements,
    extras_require={
        'security': ['pyOpenSSL >= 0.14', 'cryptography>=1.3.4'],
        'socks': ['PySocks>=1.5.6, !=1.5.7'],
        'socks:sys_platform == "win32" and python_version == "2.7"': ['win_inet_pton'],
    },
    project_urls={
        'Documentation': 'https://requests.readthedocs.io',
        'Source': 'https://github.com/psf/requests',
    },
)
```

As you can see from above, the `requires` variable at the beginning is listing all the packages that are required for `requests` to work. Graphs like the one I've showed before are generated base on those directives. This file will also have other useful info like project metadata, target python versions, classifiers, license... Digging into those is out of the scope of the post, and so is the fact that `setup.py` is bound to be replaced by something better in the future[^1].

Files like `setup.py` are used when a project is converted to a _wheel_. A wheel is the actual file you generally [^2] download whenever you install a package using `pip`. Even though the package has a `.whl` extension, it's just a zip file holding metadata about the package and the actual dependency code you'll call from your project. So while you can express which dependencies you want through `setup.py`, it is actually the wheel that will hold the list of dependencies when your package is distributed.

So now you're aware that a simple `pip install` can actually install a bunch of stuff and not only the single package you've specified. You also know that a package has ways to specify _other_ packages that are needed for it to work. But things start to get really interesting when a system needs to decide which versions of each package should be installed for a project. If underlying packages end up somehow asking for conflicting versions, what should you do?

## Dependency Resolution

By now, if you thought underlying dependencies can create all sorts of chaos if not handled properly, then you're goddam right. Let's consider the following example to illustrate some of the issues.

You want to download and install a python package called `lamb-of-god`. This package depends on two other packages, `metallica` and `megadeth`. Both of these packages depend on another package called `black-sabbath`. What if `metallica` and `megadeth` have different version requirements for `black-sabbath`? How to decide which version to install?

You're probably confused - btw, welcome to dependencies - so here's what I'm taking about:

```python
LAMB-OF-GOD 1.0 requires:
    - METALLICA >= 1.1, <2.0
    - MEGADETH >= 2.3

    METALLICA 1.2 requires:
        - BLACK-SABBATH >= 1.0, < 2.0

    MEGADETH 2.3 requires:
        - BLACK-SABBATH >= 1.3 < 2.0

# (assuming that METALLICA 1.2 and MEGADETH 2.3 are the latest available versions of those packages)
```

Considering that the latest `black-sabbath` ever released was `1.8`, we could probably say that `black-sabbath==1.8` is the right version we should use (it would match the requirements of both `metallica` and `megadeth`). But what if, while deciding to upgrade our `lamb-of-god` package on a later date, we end up with a tree that looks like this:

```python
LAMB-OF-GOD 1.1 requires:
    - METALLICA >= 1.1, <2.0
    - MEGADETH >= 2.3

    METALLICA 1.5 requires:    # new release came out, tree updated
        - BLACK-SABBATH >= 1.0, < 2.0

    MEGADETH 3.0 requires:     # new release came out, tree updated
        - BLACK-SABBATH >= 2.0 < 3.0

# (now METALLICA 1.5 and MEGADETH 3.0 are the latest available versions of those packages)
```

Can you spot the problem with those requirements? There's **no version** of `black-sabbath` that would serve both `metallica` and `megadeth` (actually in the early days I bet there wasn't a lot of things that these two bands would agree with). What we have here is a **dependency conflict** - underlying dependencies are asking for two conflicting versions of the same package. That's one of the jobs of a dependency resolution algorithm: traverse through the dependency tree, try to find versions that would meet _all_ requirements and stop the installation in case of any conflicts.

What's hard to grasp is that this might _not_ be `lamb-of-god`'s fault. Maybe the package was created before `metallica==1.5` and `megadeth==3.0` were released, and back then they would agree on which version of `black-sabbath` to use. Packages are often not maintained by the same group of developers, and errors due to misaligned dependencies are [far from being rare](https://www.google.com/search?client=firefox-b-1-d&q=python+dependency+conflict).

But why are misaligned dependencies a problem? Well, different versions of packages might contain different code and different APIs (args/functions you can use). For example, in the second tree, the `black-sabbath` that `metallica` wants might have a module called `ozzy-osbourne`, but the version `megadeth` wants replaced that module by another one called `ronnie-james-dio` (if you didn't get the joke, Black Sabbath switched singers at some point). It's not just names that can change - the code itself might be doing something completely different, _even if the function name/args stays the same_. Did you ever face a really sketchy bug that makes you want to cry on a Friday at 4:59p? This is one of the ways they come to life.

Realizing misaligned dependencies are a problem is the first step towards proper dependency management. But you know what's really fun about this whole ordeal? Older versions of `pip` would **not** have bombed with my second example! Here's a quote from [this](https://pyfound.blogspot.com/2020/03/new-pip-resolver-to-roll-out-this-year.html) blog post from March 2020, that explains some of the problems with old `pip`:


_(while talking about some of the proposed enhancements)_

- _It will reduce inconsistency: it will no longer install a combination of packages that is mutually inconsistent. At the moment, it is possible for pip to install a package which does not satisfy the declared requirements of another installed package. For example, right now, pip install "six<1.12" "virtualenv==20.0.2" does the wrong thing, “successfully” installing six==1.11, even though virtualenv==20.0.2 requires six>=1.12.0,<2 (defined here). The new resolver would, instead, outright reject installing anything if it got that input._
- _It will be stricter - if you ask pip to install two packages with incompatible requirements, it will refuse (rather than installing a broken combination, like it does now)._


The bottom line of this text is that old versions of `pip` (<20.3) will not care about conflicts: it just picks one of the two. Now of course there were ways around that (like using a better dependency management tool or being clever about the order of `pip` of operations), but there are lots of people that were/are completely unaware of this and _will end up with conflicting dependencies in their environments_. The project might still work for a while, but when dependencies get misaligned, you might end up with a time bomb in your hands.

Another issue you might face if don't tame your dependencies from the start, is that trying to fix the dependency tree _after you became used to conflicting dependencies_ can be a huge pain in the ass. I've seen the dread in some developer's eyes from miles away when they were tasked with such work.

Luckily, today there's plenty of resources you can use to get you dependencies aligned. With `pip`'s new dependency resolver[^3], odds are most new projects won't suffer as much. We also have ways to pin dependencies to certain versions once the tree has been resolved, preventing the system from having to re-solve the tree during installations (think Docker or pipelines). But is proper dependency resolution enough?

## Lockfiles

Let's say your dependency tree was resolved by a good dependency resolver algorithm and you had no conflicting dependencies. A common practice in the industry is to _lock_ those installed versions, so that future installations of you project won't have to resolve the tree again, referring to the versions locked in a file.

That's one of the use-cases for _lockfiles_: persisting into a file the list of _specific_ dependencies returned from the dependency resolver. So a potential representation of a lock file for my first `lamb-of-code==1.0` example could be:

```python
# rudimentar_example_of_pinning_dependencies.lock

metallica==1.2
megadeth==2.3
black-sabbath==1.8
```

Further installations would not try to resolve the dependency chain - they would go straight up into the list of pins and use that to install the project. Lockfiles are normally committed to the repo, so that other users / image builds / testing frameworks / pipelines can install based on that list of dependencies that should work well together. These will often be called _deterministic builds_.

However, lockfiles can give you more than just a simple list of pinned dependencies.

### Package Integrity

PyPi is an open repository and there's not a formal review process before a package gets accepted. PyPi is also a constant target of attacks by malicious individuals, trying to mess up with your system/wallet:

- https://arstechnica.com/gadgets/2021/06/counterfeit-pypi-packages-with-5000-downloads-installed-cryptominers/
- https://www.theverge.com/2021/2/10/22276857/security-researcher-repository-exploit-apple-microsoft-vulnerability
- https://www.theregister.com/2021/03/02/python_pypi_purges/

While one of the most obvious forms of attacks is [typosquatting](https://nakedsecurity.sophos.com/2017/09/19/pypi-python-repository-hit-by-typosquatting-sneak-attack/) (_slightly_ changing the name of the package, in the hopes someone makes a typo during installs), you really can't know if maintainers of a proper library had their systems hijacked, converting the _code_ (not the name) of a package to evil. The package name/version might still be the same, but the contents inside could have been changed. How do you protect your system from this?

Well, your lockfile can store a _hash_ for each one of the dependencies on the first install, which can later be used to validate if the package you're downloading has _really_ the same contents you are expecting. Hashes for packages are normally provided by the package index (like [PyPi](https://pypi.org/project/numpy/#files) - notice the "View" button at the right) or be generated locally for a dependency. `pip` [has a built-in functionality](https://pip.pypa.io/en/stable/cli/pip_install/#hash-checking-mode) to double check if the hashes you have listed for a dependency matches the hash of a new download. However, [this open github issue](https://github.com/pypa/pip/issues/4732) is a good example on how this process can be cumbersome if all you use is `pip`.

Luckily there are good dependency managers _besides_ bare `pip` that we can use with Python. Tools like `pipenv` and `pip-tools` will work on top of `pip`, adding lots of functionality (like a better workflow for lockfiles), others like `poetry` will disregard `pip` completely and re-create the whole dependency management system from the ground up. They all have pro and cons for specific functionalities, and it should be up to you to decide which one you want to use.

Personally, I've started using `poetry` a while ago, but dropped it in favor of `pipenv` after I had issues with `poetry` and private package feeds, but you should use whatever works for you. As long as you have proper dependency resolution and lockfiles with hashes, you'll be a in a better spot.

## Private Feeds

Let's say you have a good dependency management system in place, lockfiles, hashes, good dependency resolution, the whole thing. If you're reading from PyPi directly, what happens when a package you want is deleted? What if the version you need to build your production image gets hijacked? And what happens if PyPi goes down? Well, you're doomed... if you're not using a _private feeds_ for your packages.

The idea here is that you want to have better control over which versions are available to internal processes in your company, relying on a paid service with a proper SLA[^4]. Internal feeds have a simple workflow: instead of hitting a public feeds like PyPi when you do `pip install`, you configure your package manager to go a internal private feed instead. If the internal feed doesn't have the version of a package you're looking for, it goes to the public feed, downloads a copy and stores it in your company's internal artifact repository. From them on, the next download _won't_ hit the public feed anymore, only the internal one. It's a good practice to not rely on public package repositories for company products, and most organizations will have some sort of private feed configured internally.

Another major advantage of private feeds is the ability to _share internal libraries_ without disrupting the developer's workflow. So if you have `companyname-utils` package, you can publish that to your private feed and tell other developers to install them using their package managers (i.e. `poetry add companyname-utils`). You can also control which teams have access to which packages, something that can be useful depending on how large is your company and the types of projects you have.

To get more familiar with private feeds, I advise you to take a look at [Azure Artifacts](https://azure.microsoft.com/en-us/services/devops/artifacts/) or [AWS CodeArtifact](https://aws.amazon.com/codeartifact/). There are other vendors out there, but you can use those as a base for the kind of functionalities you'll get from those kind of products.

But one thing to notice here is that even by using a private **you might still be at risk**! As I've mentioned before, if the package is not in your private feed the first time you try to install it, the system will search for that package in a public repository. What if that public version is infected? And what if a vulnerability is discovered days/months/years after you've already stored a certain version of a package in your internal repository?

## Vulnerability Scans

Tons of newer packages are being published everyday, and keeping mental tabs on which versions are known to be problematic is a job in itself. So how can you, a single developer, add _even more_ protection to you dependency management controls?

Turns out there are tools out there we can use to _scan_ your dependencies periodically for known security vulnerabilities. The two I've used before are [Safety](https://aws.amazon.com/codeartifact/), and [DependaBot](https://dependabot.com/) (through it's Github integration).

Safety is nice because it's a simple CLI tool that you run against your local environment. It will raise in case of any know vulnerabilities for the current installed packages. You can also use Safety inside your build pipelines, making a successful safety scan another requirement for pipelines to pass. `pipenv` [ships with `safety`](https://pipenv-fork.readthedocs.io/en/latest/advanced.html#detection-of-security-vulnerabilities), but it's important to mention that Safety is free only for Open Source projects. If you want to use that within your company, you need to sign up for an API key.

DependaBot is cool if you use Github for code versioning, as it will do almost all of the work for you. DependaBot will periodically scan the list of dependencies you've listed inside your projects, and warn you about potential vulnerabilities. It even creates the Pull Request automatically! Sound like it's _too good to be true_, but I'm using DependaBot privately at the moment and it works like a charm. The only downside is that (as far as I know) configuring DependaBot outside of Github can be a bit time-consuming, and you might want to divert efforts towards implementing something like Safety or [PyUp](https://pyup.io/).

## Testing Against Multiple Python Versions

This step is a bit broad and not only tied to dependency management, but I had luck finding dependency issues through this method **before** they became a real headache.

When you work on a certain library, normally you'll use a base python version for coding, but consumers of your library might be using _different_ python versions. You might have noticed that every package inside PyPi has a ["Download Files" page](https://pypi.org/project/pandas/#files) in the left side menu, which might list wheels that were made for specific Python/OS versions. If you work mainly with a certain python, how can you know that the underlying dependencies that were built for _other versions_ won't explode?

My last recommendation for today will be a tool called [Tox](https://tox.readthedocs.io/en/latest/). Tox will automate the creation of virtual environments for specific python versions and let you install your project _on each of them_ to run tests. If you have a comprehensive test suit, this should give you confidence that your package won't break for the python versions you want to support. Their [basic example](https://tox.readthedocs.io/en/latest/#basic-example) speaks by itself:

```python
# content of: tox.ini , put in same dir as setup.py
[tox]
envlist = py27,py36, py38, py39

[testenv]
# install pytest in the virtualenv where commands will be executed
deps = pytest
commands =
    # NOTE: you can run any command line tool here - not just tests
    pytest
```

This is useful for dependency management in the sense that you might decide to _not_ use the lockfile you've been using for development, and let the Tox environment resolve the tree again, simulating a user installing your package in their own projects. Over time this might break, anticipating issues that a new user would face on a certain python version. If you've followed the other advices in this guide, this is unlikely to happen, but it won't hurt to exercise this on top of your regular tests for each individual python version.

Now I know this step might sound a bit too "mystic" for the uninitiated, so it's ok if you can't clearly see any value from this right now. Just keep in mind that there are **ways to be ahead of the game**, and try to prevent issues your users might face while installing your code due to problems with an underlying dependency that you _did not_ create directly.

## Review

```
- Underlying dependencies can create lots of problems if not taken into account
- What you put inside your `setup.py` (or similar) matters!
- Use proper dependency resolution whenever you can (preferably from the start of a project)
- Lockfiles are a must if you want proper dependency management
- Private feeds will build more trust in your system
- Periodical vulnerability scans for dependencies will help you against know issues
- Testing dependencies against multiple python versions can help with anticipating bug reports from users
```

Phew, this was a lot! I've learned all of this the hard way, hitting walls due to bad dependency management and having to find alternatives to improve the practice. It's also funny how some of the younger developers won't grasp how complicated dependency management can be once you have all the proper systems in place.

But, at the end of the day, I guess this is what we always strive for in software development: building systems that can take care of themselves, without relying on manual intervention. Even so, personally I think that knowing the _whys_ is something imperative towards becoming a better developer. Hopefully this post helped to demystify some of the magic that happen behind the scenes when you're aiming to proper handle dependency management.

Don't hesitate to reach out directly through github in case you see areas where I can improve. Thank you a lot for reading and I'll see you in the next one!


[^1]: [This PyCon US 2021](https://www.youtube.com/watch?v=j8iXO5VErjw) talk explain what's coming down the pipe way better than I could.

[^2]: In the past there were [eggs](https://packaging.python.org/discussions/wheel-vs-egg/), but wheels are more broadly available today. You might also notice some `.tar.gz` files inside PyPi, meaning you're building directly for source, and not a wheel targeting an specific python / os version.

[^3]: I'm not going to pretend I know a lot about those specific dependency resolution algorithms, but here's a thorough description on how never versions of `pip` are dealing with the issue: https://pip.pypa.io/en/stable/user_guide/#changes-to-the-pip-dependency-resolver-in-20-3-2020.

[^4]: [Service Level Agreement](https://en.wikipedia.org/wiki/Service-level_agreement)
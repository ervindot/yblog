---
kind:           article
published:      2014-08-16
image: /Scratch/img/blog/Safer-Haskell-Install/main.jpg
en: title: Safer Haskell Install
fr: title: Installer Haskell
author: Yann Esposito
authoruri: yannesposito.com
tags: programming
theme: brutalist
---
blogimage("main.jpg","to Haskell and Beyond!!!")

<div class="intro">

en: %tldr Install Haskell (OS X and Linux only) by pasting the following in your terminal:
fr: %tlal Pour installer Haskell (OS X et Linux) copiez/collez les lignes suivante dans un terminal :

~~~
curl https://raw.githubusercontent.com/yogsototh/install-haskell/master/install-haskell.sh | sudo zsh
~~~

en: _update (28 march 2015): I now use [Haskell LTS][lts] instead of a random stackage version._

en: If you are on windows, just download the Haskell Platform and follow
en: the instruction to use [Haskell LTS][lts].
fr: Si vous êtes sous windows, téléchargez Haskell Platform
fr: et suivez les instructions pour utiliser [Haskell LTS][lts].

en: If you want to know the why and the how; you should read the entire article.
fr: Si vous voulez savoir le pourquoi et le comment ; lisez le reste de l'article.

</div>

en: ## Why?
fr: ## Pourquoi ?

en: The main weakness of Haskell as nothing to do with the language itself but
en: with its ecosystem[^1].
fr: La plus grande faiblesse d'Haskell n'a rien à voir avec le langage en lui-même
fr: mais avec son écosystème.

en: [^1]: By ecosystem of a language I mean, the community, the tools, the documentations, the deployment environments, the businesses using the language, etc... Mainly everything that has nothing to do with the detail of a programming language but has to do on how and why we use it.
fr: [^1]: Par l'écosystème d'un langage j'entends, la communauté, les outils, la documentation, les environnements de déploiements, les entreprises qui utilisent le langage, etc... En gros tout ce qui n'a rien à voir avec les détails du langage mais ce qui a à voir avec les comment et pourquoi on l'utilise.

The main problem I'll try to address is the one known as _cabal hell_.
The community is really active in fixing the issue.
I am very confident that in less than a year this problem will be one of the past.
But to work today, I provide an install method that should reduce greatly
two effects of cabal hell:

- dependency error
- lost time in compilation (poor polar bears)

With my actual installation method, you should minimize your headache and almost
never hit a dependency error.
But there could exists some.
If you encounter any dependency error,
ask gently to the package manager to port its package to [stackage][stackage].

So to install copy/paste the following three lines in your terminal:

~~~
curl https://raw.githubusercontent.com/yogsototh/install-haskell/master/install-haskell.sh | sudo zsh
~~~

## How?

You can read the script and you will see that this is quite straightforward.

1. It downloads the latest `GHC` binary for you system and install it.
2. It does the same with the `cabal` program.
3. It updates your cabal config file to use [Haskell LTS][lts].
4. It enable profiling to libraries and executables.
5. It installs some useful binaries that might cause compilation error if not present.

As the version of libraries is fixed up until you update the [Haskell LTS][lts] version,
you should never use cabal sandbox.
That way, you will only compile each needed library once.
The compiled objects/binaries will be in your `~/.cabal` directory.

## Some Last Words

This script use the latest [Haskell LTS][lts].
So if you use this script at different dates, the Haskell LTS might have changed.

While it comes to cabal hell, some solutions are sandboxes and `nix`.
Unfortunately, sandboxes didn't worked good enough for me after some time.
Furthermore, sandboxes forces you to re-compile everything by project.
If you have three yesod projects for example it means a lot of time and CPU.
Also, `nix` didn't worked as expected on OS X.
So fixing the list of package to a stable list of them seems to me the best
pragmatic way to handle the problem today.

From my point of view, [Haskell LTS][lts] is the best step in the right direction.
The actual cabal hell problem is more a human problem than a tool problem.
This is a bias in most programmer to prefer resolve social issues using tools.
There is nothing wrong with hackage and cabal.
But for a package manager to work in a static typing language as Haskell,
packages must work all together.
This is a great strength of static typed languages that they ensure that a big
part of the API between packages are compatible.
But this make the job of package managing far more difficult than in dynamic languages.

People tend not to respect the rules in package numbering[^2].
They break their API all the time.
So we need a way to organize all of that.
And this is precisely what [Haskell LTS][lts] provide.
A set of stable packages working all together.
So if a developer break its API, it won't work anymore in stackage.
And whether the developer fix its package or all other packages upgrade their usage.
During this time, [Haskell LTS][lts] end-users will be able to develop without dependency issues.

[^2]: I myself am guilty of such behavior. It was a beginner error.

[lts]: http://www.stackage.org/lts
[stackage]: http://www.stackage.org

---

<p class="small">
[The image of the cat about to jump that I slightly edited can found here](https://www.flickr.com/photos/nesster/4198442186/)
</p>



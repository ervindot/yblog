---
kind:           article
published:      2013-03-16
image: /Scratch/img/blog/Hakyll-setup/main.png
title: Hakyll setup
author: Yann Esposito
authoruri: yannesposito.com
tags: programming, hakyll, Haskell, nanoc
theme: brutalist
---
blogimage("main.png","Main image")

<div class="intro">

%tldr How I use [hakyll](http://jaspervdj.be/hakyll).
Abbreviations, typography corrections, multi-language,
use `index.html`, etc...


</div>

This website is done with [Hakyll][hakyll].

[hakyll]: http://jaspervdj.be/hakyll

[Hakyll][hakyll] can be considered as a minimal %cms.
But more generally it is a library helping file generation.
We can view it as an advanced build system (like `make`).

From the user perspective I blog this way:

1. I open an editor (vim in my case) and edit a markdown file.
   It looks like this

``` markdown
A First Level Header
====================

A Second Level Header
---------------------

Who would cross the Bridge of Death must answer me
these questions three, ere the other side he see.
This is just a regular paragraph.

Ask me the questions, bridgekeeper. I am not afraid.

### Header 3

> This is a blockquote.
>
> This is the second paragraph in the blockquote.
>
> ## This is an H2 in a blockquote
```

2. I open a browser and reload time to time to see the change.
3. Once I finished I've written a very minimal script which mainly do a `git push`.
   My blog is hosted on [github](http://github.com).

Being short sighted one could reduce the role of Hakyll to:

> create (resp. update) %html file
> when I create (resp. change) a markdown file.

While it sounds easy, there are a lot of hidden details:

- Add metadatas like keywords.
- Create an archive page containing a list of all the posts.
- Deal with static files.
- Creating an %rss feed.
- Filter the content with some function.
- Dealing with dependencies.

The work of Hakyll is to help you with these.
But let's start with the basic concepts.

## The concepts and syntax

blogimage("overview.png","Overview")

For each file you create, you have to provide:

- a destination path
- a list of content filters.

First, let's start with the simplest case: static files
(images, fonts, etc...).
Generally, you have a source directory (here is the current directory)
and a destination directory ``_site``.

The Hakyll code is:

``` haskell
-- for each file in the static directory
match "static/*" do
  -- don't change its name nor directory
  route   idRoute
  -- don't change its content
  compile copyFileCompiler
```

This program will copy ``static/foo.jpg`` to ``_site/static/foo.jpg``.
I concede this is a bit overkill for a simple `cp`.
Now how to write a markdown file and generate an %html one?

``` haskell
-- for each file with md extension in the "posts/" directory
match "posts/*.md" do
  -- change its extension to html
  route $ setExtension "html"
  -- use pandoc library to compile the markdown content into html
  compile $ pandocCompiler
```

If you create a file ``posts/foo.md``,
it will create a file ```_site/posts/foo.html```.

If the file ``posts/foo.md`` contains

``` markdown
# Cthulhu

ph'nglui mglw'nafh Cthulhu R'lyeh wgah'nagl fhtagn
```

the file ``_site/posts/foo.html``, will contain

``` html
<h1>Cthulhu</h1>
<p>ph'nglui mglw'nafh Cthulhu R'lyeh wgah'nagl fhtagn</p>
```

But horror! ```_site/posts/cthulhu.html``` is not a complete %html file.
It doesn't have any header nor footer, etc...
This is where you use templates.
I simply add a new directive in the compile block.

``` haskell
match "posts/*.md" do
  route $ setExtension "html"
  compile $ pandocCompiler
    -- use the template with the current content
    {-hi-}>>= loadAndApplyTemplate "templates/post.html" defaultContext{-/hi-}
```

Now if ``templates/posts.html`` contains:

``` html
<html>
  <head>
    <title>How could I get the title?</title>
  </head>
  <body>
    {-hi-}$body${-/hi-}
  </body>
</html>
```

our `cthulhu.html` contains (indentation added for readability):

``` html
<html>
  <head>
    <title>How could I get the title?</title>
  </head>
  <body>
    {-hi-}<h1>Cthulhu</h1>{-/hi-}
    {-hi-}<p>ph'nglui mglw'nafh Cthulhu R'lyeh wgah'nagl fhtagn</p>{-/hi-}
  </body>
</html>
```

See, it's easy
But we have a problem. How could we change the title
or add keywords?

The solution is to use `Context`s.
For this, we first need to add some _metadatas_ to our markdown[^1].

[^1]: We could also add the metadatas in an external file (`foo.md.metadata`).

``` markdown
{-hi-}--- {-/hi-}
{-hi-}title: Cthulhu{-/hi-}
{-hi-}--- {-/hi-}
# Cthulhu

ph'nglui mglw'nafh Cthulhu R'lyeh wgah'nagl fhtagn
```

And modify slightly our template:

``` html
<html>
  <head>
    <title>{-hi-}$title${-/hi-}</title>
  </head>
  <body>
    $body$
  </body>
</html>
```

As Sir Robin said just before dying before the Bridge of Death:

> **"That's EASY!"**
>
> -- <cite>Sir Robin,
> the Not-Quite-So-Brave-As-Sir-Lancelot</cite>



## Real customization

Now that we understand the basic functionality.
How to:

- use SASS?
- add keywords?
- simplify %url?
- create an archive page?
- create an %rss feed?
- filter the content?
- add abbreviations support?
- manage two languages?

### Use SASS

That's easy.
Simply call the executable using `unixFilter`.
Of course you'll have to install SASS (`gem install sass`).
And we also use compressCss to gain some space.

``` haskell
match "css/*" $ do
    route   $ setExtension "css"
    compile $ getResourceString >>=
              withItemBody (unixFilter "sass" ["--trace"]) >>=
              return . fmap compressCss
```

### Add keywords

In order to help to reference your website on the web, it is nice
to add some keywords as meta datas to your %html page.

``` html
<meta name="keywords"
      content="Cthulhu, Yog-Sothoth, Shub-Niggurath">
```

In order to add keywords, we could not directly use the markdown metadatas.
Because, without any, there should be any meta tag in the %html.

An easy answer is to create a `Context` that will contains the meta tag.

``` haskell
-- metaKeywordContext will return a Context containing a String
metaKeywordContext :: Context String
-- can be reached using $metaKeywords$ in the templates
-- Use the current item (markdown file)
metaKeywordContext = field "metaKeywords" $ \item -> do
  -- tags contains the content of the "tags" metadata
  -- inside the item (understand the source)
  tags <- getMetadataField (itemIdentifier item) "tags"
  -- if tags is empty return an empty string
  -- in the other case return
  --   <meta name="keywords" content="$tags$">
  return $ maybe "" showMetaTags tags
    where
      showMetaTags t = "<meta name=\"keywords\" content=\""
                       ++ t ++ "\">\n"
```

Then we pass this `Context` to the `loadAndApplyTemplate` function:

``` haskell
match "posts/*.md" do
  route $ setExtension "html"
  compile $ pandocCompiler
    -- use the template with the current content
    >>= loadAndApplyTemplate "templates/post.html"
            (defaultContext {-hi-}<> metaKeywordContext{-/hi-})
```

> ☞ Here are the imports I use for this tutorial.
>
> ``` haskell
> {-# LANGUAGE OverloadedStrings #-}
> import           Control.Monad          (forM,forM_)
> import           Data.List              (sortBy,isInfixOf)
> import           Data.Monoid            ((<>),mconcat)
> import           Data.Ord               (comparing)
> import           Hakyll
> import           System.Locale          (defaultTimeLocale)
> import           System.FilePath.Posix  (takeBaseName,takeDirectory
>                                          ,(</>),splitFileName)
> ```



### Simplify %url

What I mean is to use url of the form:

```
http://domain.name/post/title-of-the-post/
```

I prefer this than having to add file with `.html` extension.
We have to change the default Hakyll route behavior.
We create another function `niceRoute`.

``` haskell
-- replace a foo/bar.md by foo/bar/index.html
-- this way the url looks like: foo/bar in most browsers
niceRoute :: Routes
niceRoute = customRoute createIndexRoute
  where
    createIndexRoute ident =
        takeDirectory p </> takeBaseName p </> "index.html"
    where p=toFilePath ident
```

Not too difficult. But! There might be a problem.
What if there is a `foo/index.html` link instead of a clean `foo/` in some content?

Very simple, we simply remove all `/index.html` to all our links.

``` haskell
-- replace url of the form foo/bar/index.html by foo/bar
removeIndexHtml :: Item String -> Compiler (Item String)
removeIndexHtml item = return $ fmap (withUrls removeIndexStr) item
  where
    removeIndexStr :: String -> String
    removeIndexStr url = case splitFileName url of
        (dir, "index.html") | isLocal dir -> dir
        _                                 -> url
        where isLocal uri = not (isInfixOf "://" uri)
```

And we apply this filter at the end of our compilation

``` haskell
match "posts/*.md" do
  {-hi-}route $ niceRoute{-/hi-}
  compile $ pandocCompiler
    -- use the template with the current content
    >>= loadAndApplyTemplate "templates/post.html" defaultContext
    {-hi-}>>= removeIndexHtml{-/hi-}
```

### Create an archive page

Creating an archive start to be difficult.
There is an example in the default Hakyll example.
Unfortunately, it assumes all posts prefix their name with a date
like in `2013-03-20-My-New-Post.md`.

I migrated from an older blog and didn't want to change my %url.
Also I prefer not to use any filename convention.
Therefore, I add the date information in the metadata `published`.
And the solution is here:

``` haskell
match "archive.md" $ do
  route $ niceRoute
  compile $ do
    body <- getResourceBody
    return $ renderPandoc body
      {-hi-}>>= loadAndApplyTemplate "templates/archive.html" archiveCtx{-/hi-}
      >>= loadAndApplyTemplate {-hi-}"templates/base.html"{-/hi-} defaultContext
      >>= removeIndexHtml
```

Where `templates/archive.html` contains

``` html
$body$

<ul>
    $posts$
</ul>
```

And `base.html` is a standard template (simpler than `post.html`).

`archiveCtx` provide a context containing an %html representation
of a list of posts in the metadata named `posts`.
It will be used in the `templates/archive.html` file with `$posts$`.

``` haskell
archiveCtx =
  defaultContext <>
  metaKeywordContext <>
  {-hi-}field "posts" (\_ -> postList createdFirst){-/hi-}
```

`postList` returns an %html representation of a list of posts
given an Item sort function.
The representation will apply a minimal template on all posts.
Then it concatenate all the results.
The template is `post-item.html`:

``` html
<li><a href="$url$">$published$ - $title$</a></li>
```

Here is how it is done:

``` haskell
postList :: [Item String] -> Compiler [Item String]
            -> Compiler String
postList sortFilter = do
    -- sorted posts
    posts   <- loadAll "post/*" >>= sortFilter
    itemTpl <- loadBody "templates/post-item.html"
    -- we apply the template to all post
    -- and we concatenate the result.
    -- list is a string
    list    <- applyTemplateList itemTpl defaultContext posts
    return list
```

`createdFirst` sort a list of item and put it inside `Compiler` context.
We need to be in the `Compiler` context to access metadatas.

``` haskell
createdFirst :: [Item String] -> Compiler [Item String]
createdFirst items = do
  -- itemsWithTime is a list of couple (date,item)
  itemsWithTime <- forM items $ \item -> do
    -- getItemUTC will look for the metadata "published" or "date"
    -- then it will try to get the date from some standard formats
    utc <- getItemUTC defaultTimeLocale $ itemIdentifier item
    return (utc,item)
  -- we return a sorted item list
  return $ map snd $ reverse $ sortBy (comparing fst) itemsWithTime
```

It wasn't so easy.
But it works pretty well.

### Create an %rss feed

To create an %rss feed, we have to:

- select only the lasts posts.
- generate partially rendered posts (no css, js, etc...)

We could then render the posts twice.
One for %html rendering and another time for %rss.
Remark we need to generate the %rss version to create the %html one.

One of the great feature of Hakyll is to be able to save snapshots.
Here is how:

``` haskell
match "posts/*.md" do
  route $ setExtension "html"
  compile $ pandocCompiler
    -- save a snapshot to be used later in rss generation
    {-hi-}>>= saveSnapshot "content"{-/hi-}
    >>= loadAndApplyTemplate "templates/post.html" defaultContext
```

Now for each post there is a snapshot named "content" associated.
The snapshots are created before applying a template and after applying pandoc.
Furthermore feed don't need a source markdown file.
Then we create a new file from no one.
Instead of using `match`, we use `create`:

``` haskell
create ["feed.xml"] $ do
      route idRoute
      compile $ do
        -- load all "content" snapshots of all posts
        loadAllSnapshots "posts/*" "content"
        -- take the latest 10
        >>= (fmap (take 10)) . createdFirst
        -- renderAntom feed using some configuration
        >>= renderAtom feedConfiguration feedCtx
      where
        feedCtx :: Context String
        feedCtx =  defaultContext <>
                   -- $description$ will render as the post body
                   {-hi-}bodyField "description"{-/hi-}
```

The `feedConfiguration` contains some general informations about the feed.

``` haskell
feedConfiguration :: FeedConfiguration
feedConfiguration = FeedConfiguration
  { feedTitle = "Great Old Ones"
  , feedDescription = "This feed provide information about Great Old Ones"
  , feedAuthorName = "Abdul Alhazred"
  , feedAuthorEmail = "abdul.alhazred@great-old-ones.com"
  , feedRoot = "http://great-old-ones.com"
  }
```

Great idea certainly steal from [nanoc][nanoc] (my previous blog engine)!

[nanoc]: http://nanoc.ws

### Filter the content

As I just said, [nanoc][nanoc] was my preceding blog engine.
It is written in Ruby and as Hakyll, it is quite awesome.
And one thing Ruby does more naturally than Haskell is regular expressions.
I had a _lot_ of filters in nanoc.
I lost some because I don't use them much.
But I wanted to keep some.
Generally, filtering the content is just a way to apply
to the body a function of type `String -> String`.

Also we generally want prefilters (to filter the markdown)
and postfilters (to filter the %html after the pandoc compilation).

Here is how I do it:

``` haskell
markdownPostBehavior = do
  route $ niceRoute
  compile $ do
    body <- getResourceBody
    {-hi-}prefilteredText <- return $ (fmap preFilters body){-/hi-}
    {-hi-}return $ renderPandoc prefilteredText{-/hi-}
    {-hi-}>>= applyFilter postFilters{-/hi-}
    >>= saveSnapshot "content"
    >>= loadAndApplyTemplate "templates/post.html"    yContext
    >>= loadAndApplyTemplate "templates/boilerplate.html" yContext
    >>= relativizeUrls
    >>= removeIndexHtml
```

Where

``` haskell
applyFilter strfilter str = return $ (fmap $ strfilter) str
preFilters :: String -> String
postFilters :: String -> String
```

Now we have a simple way to filter the content.
Let's augment the markdown ability.

### Add abbreviations support

Comparing to %latex, a very annoying markdown limitation
is the lack of abbreviations.

Fortunately we can filter our content.
And here is the filter I use:

``` haskell
abbreviationFilter :: String -> String
abbreviationFilter = replaceAll "%[a-zA-Z0-9_]*" newnaming
  where
    newnaming matched = case M.lookup (tail matched) abbreviations of
                          Nothing -> matched
                          Just v -> v
abbreviations :: Map String String
abbreviations = M.fromList
    [ ("html", "<span class=\"sc\">html</span>")
    , ("css", "<span class=\"sc\">css</span>")
    , ("svg", "<span class=\"sc\">svg</span>")
    , ("xml", "<span class=\"sc\">xml</span>")
    , ("xslt", "<span class=\"sc\">xslt</span>") ]
```

It will search for all string starting by '%' and it will search
in the `Map` if there is a corresponding abbreviation.
If there is one, we replace the content.
Otherwise we do nothing.

Do you really believe I type

``` {.html .wrap}
%latex
```

each time I write %latex?

### Manage two languages

Generally I write my post in English and French.
And this is more difficult than it appears.
For example, I need to filter the language in order to get
the right list of posts.
I also use some words in the templates and I want them to be translated.

``` html
<a href="$otherLanguagePath$"
	onclick="setLanguage('$otherlanguage$')">
	{-hi-}$changeLanguage${-/hi-} </a>
```

First I create a Map containing all translations.

``` haskell
data Trad = Trad { frTrad :: String, enTrad :: String }

trads :: Map String Trad
trads = M.fromList $ map toTrad [
   ("changeLanguage",
      ("English"
      , "Français"))
  ,("switchCss",
      ("Changer de theme"
      ,"Change Theme"))
  ,("socialPrivacy",
      ("Ces liens sociaux préservent votre vie privée"
      ,"These social sharing links preserve your privacy"))
  ]
  where
    toTrad (key,(french,english)) =
      (key, Trad { frTrad = french , enTrad = english })
```

Then I create a context for all key:

``` haskell
tradsContext :: Context a
tradsContext = mconcat (map addTrad (M.keys trads))
  where
    addTrad :: String -> Context a
    addTrad name =
      field name $ \item -> do
          lang <- itemLang item
          case M.lookup name trads of
              Just (Trad lmap) -> case M.lookup (L lang) lmap of
                          Just tr -> return tr
                          Nothing -> return ("NO TRANSLATION FOR " ++ name)
              Nothing -> return ("NO TRANSLATION FOR " ++ name)
```
## Conclusion

The full code is [here](http://github.com/yogsototh/yblog.git).
And except from the main file, I use literate Haskell.
This way the code should be easier to understand.

If you want to know why I switched from nanoc:

My preceding nanoc website was a bit too messy.
So much in fact, that the dependency system
recompiled the entire website for any change.

So I had to do something about it.
I had two choices:

1. Correct my old code (in Ruby)
2. Duplicate the core functionalities with Hakyll (in Haskell)

I added too much functionalities in my nanoc system.
Starting from scratch (almost) remove efficiently a lot of unused crap.

So far I am very happy with the switch.
A complete build is about 4x faster.
I didn't broke the dependency system this time.
As soon as I modify and save the markdown source,
I can reload the page in the browser.

I removed a lot of feature thought.
Some of them will be difficult to achieve with Hakyll.
A typical example:

In nanoc I could take a file like this as source:

``` markdown
# Title

content

<code file="foo.hs">
main = putStrLn "Cthulhu!"
</code>
```

And it will create a file `foo.hs` which could then be downloaded.

``` html
<h1>Title</h1>

<p>content</p>

<a href="code/foo.hs">Download foo.hs</a>
<pre><code>main = putStrLn "Cthulhu!"</code></pre>
```

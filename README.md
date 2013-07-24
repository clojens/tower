**[API docs](http://ptaoussanis.github.io/tower/)** | **[CHANGELOG](https://github.com/ptaoussanis/tower/blob/master/CHANGELOG.md)** | [contact & contributing](#contact--contributing) | [other Clojure libs](https://www.taoensso.com/clojure-libraries) | [Twitter](https://twitter.com/#!/ptaoussanis) | current [semantic](http://semver.org/) version:

# Tower, a Clojure i18n & L10n library

The Java platform provides some very capable tools for writing internationalized applications. Unfortunately, they can be... cumbersome. We can do much better in Clojure.

Tower's an attempt to present a **simple, idiomatic internationalization and localization** story for Clojure.

It wraps standard Java functionality where possible - with warm, fuzzy, functional love. It does go in its own direction for translation but I suspect you'll like it so give it a chance!

## Features

What's in the box™? Tower comes with a rich packet of features under the hood and continues to be further developed:
* Small, uncomplicated **all-Clojure** library.
* Ridiculously simple, high-performance wrappers for standard Java **localization features**.
* Rails-like, all-Clojure **translation function**.
* **Simple, map-based** translation dictionary format. No XML or resource files!
* Automatic dev-mode **dictionary reloading** for rapid REPL development.
* Seamless **markdown support** for translators.
* **Ring middleware**.

## Prerequisites

The latest stable release of Tower needs `Clojure 1.4+` installed, as of Tower version `1.7.0+`.

To pull in the dependencies, one obviously needs a computer with working internet connection.

## Release information

If you know your way around the Clojure landscape, you'll know enough to use the references following below. If not, continue reading for a beginners mini-tutorial on setting up your first Clojure project, as an upstep to the internationalization of your program.

### Latest stable release

```clojure
[com.taoensso/tower "1.7.1"]
```

### Development version API v2 **breaking**

If you are the adventurous type, use the following dependency in your project, but please make sure to read the [development notes][devnotes]:

```clojure
[com.taoensso/tower "2.0.0-beta1"]
```

## Setup instructions

Tower [artifacts][artifacts] are [released to [Clojars][release] and it's the default repository for [Leiningen][lein], but you can use it with other build tools like [Maven][mvn] exclusively as well.

### Installation using Leiningen

Leiningen uses Clojars out of the box. Therefore all that suffices is to append the required Tower release information to the `:dependency` key of your `project.clj` leiningen 2 file.

After leiningen is installed, you may use e.g. `lein new mytowerapp` to generate a clean project structure and append to the generated `project.clj` file e.g.:

```clojure
(defproject mytowerapp "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :dependencies [
   [org.clojure/clojure "1.5.1"]
   [com.taoensso/tower "1.7.1"] ; <- change version as needed
   ])
```

### Installation using Apache Maven

If you are using Maven as your sole build tool, add the following repository definition to your build configuration details in the `[pom.xml][pom]` file as a child of the `<repositories>` node:

```xml
<repository>
      <id>clojars</id>
      <url>https://clojars.org/repo/</url>
      <snapshots>
            <enabled>true</enabled>
      </snapshots>
      <releases>
            <enabled>true</enabled>
      </releases>
</repository>
```

Next add the required dependency to the `<dependencies>` node of the `pom.xml` file, change the version number as applicable:

```xml
<dependency>
      <groupId>com.taoensso</groupId>
      <artifactId>tower</artifactId>
      <version>1.7.1</version>
</dependency>
```

## Usage

Create a new file or append to existing namespace definitions, e.g. `mytowerapp/src/mytowerapp/core.clj` the `:use` directive to include and load the Tower namespace.

```clojure
(ns mytowerapp.core
 (:use [taoensso.tower]))
```

## Configuration

The `t` function handles translations in Tower API `v1`. You give it a config map which includes your dictionary and you're ready to go:

```clojure
(def my-tconfig
  {:dev-mode? true
   :fallback-locale :en
   :dictionary
   {:en        {:example  {:foo ":en :example/foo text"
                           :foo_note  "Hello translator, please do x"
                           :bar {:baz ":en :example.bar/baz text"}
                           :greeting  "Hello {0}, how are you?"
                           :with-markdown "<tag>**strong**</tag>"
                           :with-exclaim! "<tag>**strong**</tag>"
                           :greeting-alias :example/greeting
                           :baz-alias      :example.bar/baz}
                 :missing  "<Translation missing: {0}>"}
    :en-US      {:example {:foo ":en-US :example/foo text"}}
    :en-US-var1 {:example {:foo ":en-US-var1 :example/foo text"}}}

   :log-missing-translation-fn (fn [{:keys [dev-mode? locale ks]}]
                                #_ "... other stuff to be added here"
                                )})
```

;;; Translation strings are escaped and parsed as inline Markdown:
(t :en my-tconfig :example/with-markdown) ;=> "&lt;tag&gt;<strong>strong</strong>&lt;/tag&gt;"
(t :en my-tconfig :example/with-exclaim)  ;=> "<tag>**strong**</tag>" ; Notice no "!" suffix here, only in dictionary map

(t :en-US my-tconfig :example/foo)              ;=> ":en-US :example/foo text"
(t :en    my-tconfig :example/foo)              ;=> ":en :example/foo text"
(t :en    my-tconfig :example/greeting "Steve") ;=> "Hello Steve, how are you?"
```

It's simple to get started, but there's a number of advanced features for if/when you need them:

**Loading dictionaries from disk/resources**: Just use a string for the `:dictionary` value in your config map. For example, `:dictionary "my-dictionary.clj"` will load a dictionary map from the "my-dictionary.clj" file on your classpath or one of Leiningen's resource paths (e.g. `resources/`).

**Reloading dictionaries on modification**: Just make sure `:dev-mode? true` is in your config, and you're good to go!

**Scoping translations**: Use `with-tscope` if you're calling `t` repeatedly within a specific translation-namespace context:

```clojure
(with-tscope :example
  [(t :en my-tconfig :foo)
   (t :en my-tconfig :bar/baz)]) ;=> [":en :example/foo text" ":en :example.bar/baz text"]
```

**Missing translations**: These are handled gracefully. `(t :en-US my-tconfig :example/foo)` will search for a translation as follows:
  1. `:example/foo` in the `:en-US` locale.
  2. `:example/foo` in the `:en` locale.
  3. `:example/foo` in the dictionary's default locale.
  4. `:missing` in any of the above locales.

You can also specify fallback keys that'll be tried before other locales. `(t :en-US my-tconfig [:example/foo :example/bar]))` searches:
  1. `:example/foo` in the `:en-US` locale.
  2. `:example/bar` in the `:en-US` locale.
  3. `:example/foo` in the `:en` locale.
  4. `:example/bar` in the `:en` locale.
  5. `:example/foo` in the default locale.
  6. `:example/bar` in the default locale.
  7. `:missing` in any of the above locales.

In all cases, translation requests are logged upon fallback to default locale or :missing key.

### Localization

Check out `fmt`, `parse`, `lsort`, `fmt-str`, `fmt-msg`:
```clojure
(tower/fmt   :en-ZA 200       :currency)        ;=> "R 200.00"
(tower/fmt   :en-US 200       :currency)        ;=> "$200.00"
(tower/parse :en-US "$200.00" :currency)        ;=> 200
(tower/fmt :de-DE 2000.1 :number)               ;=> "2.000,1"
(tower/fmt :de-DE (java.util.Date.))            ;=> "12.06.2012"
(tower/fmt :de-DE (java.util.Date.) :date-long) ;=> "12. Juni 2012"
(tower/fmt :de-DE (java.util.Date.) :dt-long)   ;=> "12 giugno 2012 16.48.01 ICT"
(tower/lsort :pl ["Warsaw" "Kraków" "Łódź" "Wrocław" "Poznań"])
;=> ("Kraków" "Łódź" "Poznań" "Warsaw" "Wrocław")

(mapv #(tower/fmt-msg :de "{0,choice,0#no cats|1#one cat|1<{0,number} cats}" %)
(range 5)) ;=> ["no cats" "one cat" "2 cats" "3 cats" "4 cats"]
```

Yes, seriously- it's that simple. See the appropriate docstrings for details.

### Country and languages names, timezones, etc.

Check out `countries`, `languages`, and `timezones`.

### Ring middleware

Quickly internationalize your Ring web apps by adding `tower.ring/wrap-tower-middleware` to your middleware stack.

It'll select the best available locale for each request then establish a thread-local locale binding with `tower/*locale*`, and add `:locale` and `:t` request keys.

See the docstring for details.

## This project supports the CDS and ![ClojureWerkz](https://raw.github.com/clojurewerkz/clojurewerkz.org/master/assets/images/logos/clojurewerkz_long_h_50.png) goals

  * [CDS](http://clojure-doc.org/), the **Clojure Documentation Site**, is a **contributer-friendly** community project aimed at producing top-notch, **beginner-friendly** Clojure tutorials and documentation. Awesome resource.

  * [ClojureWerkz](http://clojurewerkz.org/) is a growing collection of open-source, **batteries-included Clojure libraries** that emphasise modern targets, great documentation, and thorough testing. They've got a ton of great stuff, check 'em out!

## Contact & contribution

Please use the [project's GitHub issues page](https://github.com/ptaoussanis/tower/issues) for project questions/comments/suggestions/whatever **(pull requests welcome!)**.

We are very open to ideas if you have any!

Otherwise reach me (Peter Taoussanis) at [taoensso.com](https://www.taoensso.com) or on Twitter ([@ptaoussanis](https://twitter.com/#!/ptaoussanis)). Cheers!

## Glossary of Terms

### Artifacts

An artifact is a file, usually a JAR, that gets deployed to a Maven repository.

A [Maven][mvn] build produces one or more artifacts, such as a compiled JAR and a "sources" JAR.

Each artifact has a group ID (usually a reversed domain name, like `com.example.foo`),
an artifact ID (just a name), and a version string. The three together uniquely identify the artifact.

A project's dependencies are specified as artifacts.

## License

Copyright &copy; 2012, 2013 Peter Taoussanis. Distributed under the [Eclipse Public License](http://www.eclipse.org/legal/epl-v10.html), the same as Clojure.

[artifacts]: <#artifacts>
[mvn]: <http://en.wikipedia.org/wiki/Apache_Maven>
[release]: <https://clojars.org/com.taoensso/tower>
[pom]: <http://maven.apache.org/guides/introduction/introduction-to-the-pom.html#What_is_a_POM>
[devnotes]: <#development-notes>
[lein]: <https://github.com/technomancy/leiningen>
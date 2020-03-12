---
title: Publishing Cucumber reports using Travis and GitHub pages
date: 2020-03-12 02:00:00
lastmod: 2020-02-12 02:00:00
description: >-
  A guide to publishing Cucumber reports using Travis CI and GitHub pages
layout: post
---

Recently I've been working on an automated testing framework for journey planners and ticketing systems. My tests are written in Cucumber and I wanted to find a way to automatically publish the test report to a website. With Travis CI and GitHub this is remarkably easy.
{: .article-headline}

# Generating the reports

Before getting started, I would highly recommend using [Damian Szczepanik's cucumber reports](https://github.com/damianszczepanik/cucumber-reporting) as they are much more legible than the default HTML reports:

![cucumber-reports](/asset/img/publishing-cucumber-reports-with-travis/feature-overview.png){: .img-responsive }

There is a handy [guide](https://www.jvt.me/posts/2019/04/07/prettier-cucumber-jvm-html-reports/) on how to them up, but the short version is to add the dependency and configure your test runner to use the plugin:

```groovy

dependencies {
    // ... other dependencies omitted
    testImplementation("io.cucumber:cucumber-java:5.1.3")
    testImplementation("io.cucumber:cucumber-junit:5.1.3")
    testImplementation("de.monochromata.cucumber:reporting-plugin:4.0.29")
}

configurations {
    cucumberRuntime {
        extendsFrom testImplementation
    }
}

task cucumber() {
    dependsOn assemble, compileJava
    doLast {
        javaexec {
            main = "io.cucumber.core.cli.Main"
            classpath = configurations.cucumberRuntime + sourceSets.main.output + sourceSets.test.output
            args = [
                '--plugin', 'de.monochromata.cucumber.report.PrettyReports:build/reports/cucumber',
                '--glue', 'io.ljn.jp.test.runner',
                'src/main/resources'
            ]
        }
    }
}

```

Now when the cucumber task is run the report will be in the `build/reports/cucumber/cucumber-html-reports` folder.

# Publishing

Having generated the report we can now plug it in to our Travis CI pipeline so that the results get published to a website.

Before doing this you will need to generate a GitHub personal access token. Travis CI provide [instructions on how to do this](https://help.github.com/articles/creating-an-access-token-for-command-line-use/). Once you've generated your token, add it to the environment variables for your build as `GITHUB_TOKEN`.

With the GitHub access token is set up we need to add a deploy step to `.travis.yml`:

```yaml
script:
  - ./gradlew check
  - ./gradlew cucumber || echo "done"
before_deploy:
  - cp build/reports/cucumber/cucumber-html-reports/overview-features.html build/reports/cucumber/cucumber-html-reports/index.html
deploy:
  local_dir: build/reports/cucumber/cucumber-html-reports
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  keep_history: true
  on:
    branch: master
```

There are a couple of things to note in there.

First, `./gradlew cucumber || echo "done"` will avoid breaking the build if the cucumber tests fail. Travis will not run the deploy job if the build fails, adding `|| echo "done"` will force an exit code of 0. Depending on the context you may or may not want this.

Second, the `before_deploy` step is copying the `overview-features.html` file to `index.html` so the published website has a homepage.

Now if you go to your GitHub repository settings an enable GitHub pages you will see the report published the next time you run a build:

![website](/asset/img/publishing-cucumber-reports-with-travis/website.png){: .img-responsive }

# Automating

You can go a step further and set up a [Travis Cron](https://docs.travis-ci.com/user/cron-jobs/) to automatically run the build every day to regenerate the report.

![cron](https://docs.travis-ci.com/images/cron-section.png){: .img-responsive }

Then you'll have fresh Cucumber reports every day.

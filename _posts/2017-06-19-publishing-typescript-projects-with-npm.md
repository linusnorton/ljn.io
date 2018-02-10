---
title: How to publish TypeScript projects with NPM
date: 2017-06-19 18:00
lastmod: 2017-06-19 18:00
description: Publishing a TypeScript project is not a straight forward as it might seem. This is the how, what and why of TypeScript / NPM publishing.
layout: post
---

Over the last 12 months I've been toying with and ultimately re-writing a lot of my projects in TypeScript. It's been a remarkably smooth learning curb and I must say I'm really enjoying it, but whenever you pick up new things there are bound to be some issues along the way. The first head scratching moment I encountered was not actually an issue with the language itself but how to distribute a TypeScript library as an NPM package.

```

  TLDR; When publishing a TypeScript project on NPM, run a prepublish script to create the JavaScript
  and TypeScript definition files and publish those files instead of your TypeScript files.

```

As the TLDR above indicates, when publishing a TypeScript library with NPM, you do not typically distribute the .ts files. At first this seemed a bit odd to me, particularly as my use case was for TypeScript only projects. 

## Why, why, why

It turns out there are some good reasons not to publish .ts files:
 - ts-node will not compile TS in node_modules
 - different TypeScript versions
 - different compiler options	

Essentially the compiler options have a huge impact on whether or not the code will compile and the JS generated. If TypeScript files are included directly from a library there is no mechanism for the compiler to pick up the compiler options of that project. In theory the compiler could reload the tsconfig.json for each depdencency but that would slow it down significantly and it is also unnecessary if the library is compiled ahead of time.

## How to do it, the longer version

If you add an [npm prepublishOnly](https://docs.npmjs.com/misc/scripts) script that compiles the project into to a specific folder, e.g.

```
  "scripts": {
    "run": "ts-node ./src/index.ts",
    "prepublishOnly": "tsc -p ./ --outDir dist/"
  },
```

then you can set the normal npm package.json properties relative to that directory:

```
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
```

Other TypeScript projects can then use your d.ts files instead of the actual TypeScript. Technically speaking if your main .d.ts file is called index.d.ts you do not need to add that option in package.json but [it is still recommended](https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html).

One minor annoyance from this is that the dist files are now inside your package, and potentially inside your git repository. Best practice typically says that build artifacts shouldn't be placed inside the git repository. The problem is that if you add the `./dist/` folder to your `.gitignore` file npm will also ignore it!

The solution is to add a separate `.npmignore` file. In your `.gitignore` you add the `./dist/` folder so it is not picked up in git and in the `.npmignore` you add the directory with the `.ts` files so they are not included in the package. 

## Yarn

At the moment the `yarn publish` command does not run the prepublish script. There is an [open bug](https://github.com/yarnpkg/yarn/issues/1671) for this.

## Example

If you would like to see an example of this in action, there is one [in one of my projects](https://github.com/open-track/dtd2mysql/blob/a2bb42f161945b570a33c189c12212961e4d7771/package.json).

Know a better way? Please let me know! I didn't really find another "this is how you publish TypeScript" guides out there. It was more from picking up best practice from other projects out there and some guidelines from ts-node.

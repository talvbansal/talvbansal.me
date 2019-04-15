---
title: Adding eslint to your Laravel application
date: 2018-02-04 15:56:44
tags:
- Eslint
- Laravel
- Javascript
- Ecmascript
- Tooling
- Gitlab
- CI/CD

autoThumbnailImage: yes
coverImage: ./portugal-tram.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: portugal-tram.jpg
thumbnailImagePosition: right

---
## Introduction

My latest projects at work have evolved from being PHP / Laravel applications with sprinklings of Vue JS on the front end to being almost entirely Vue JS components driving the frontend with Laravel backends.

Whilst laravel ships with [Laravel-Mix](https://laravel.com/docs/master/mix) to make webpack / asset compilation easier it doesn't ship with anyway to lint your javascript.

Luckily adding [eslint](https://eslint.org/) and defining rules for your `.js` and `.vue` files to follow is pretty straight forward.

<!-- more -->

## Getting Started

Whilst you can install eslint globally on your system I'd recommend adding it on a per-project basis so that you can include running eslint as a build step within your CI/CD process.

Add eslint to your projects package.json file.

```bash
# Yarn
yarn add -D eslint

# NPM
npm install eslint --save-dev
```

Create a base configuration

```bash
./node_modules/.bin/eslint --init
```
The above command will ask you a number of questions for my laravel projects I use the following:

```bash
Are you using ECMAScript 6 features? Yes
Are you using ES6 modules? Yes
Where will your code run? Browser
Do you use CommonJS? Yes
Do you use JSX? No
What style of indentation do you use? Tabs
What quotes do you use for strings? Double 
What line endings do you use? Unix
Do you require semicolons? Yes
What format do you want your config file to be in? JSON
```

You'll end up with a new file called `.eslintr.json` within your project root that has the following content generated as a result of the responses to the questions above.

```javascript
{
    "env": {
        "browser": true,
        "commonjs": true,
        "es6": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "sourceType": "module"
    },
    "rules": {
        "indent": [
            "error",
            "tab"
        ],
        "linebreak-style": [
            "error",
            "unix"
        ],
        "quotes": [
            "error",
            "double"
        ],
        "semi": [
            "error",
            "always"
        ]
    }
}
```

At this point eslint is now set up and ready to be used in your project.

```bash
./node_modules/.bin/eslint ./resources/assets/js/

```

Running the above should give you some sort output like the following (unless you have JS that adheres to all of the standards of course!):

```bash
/resources/assets/js/app.js
   8:9   error  Strings must use doublequote                       quotes
  16:1   error  'Vue' is not defined                               no-undef
  16:15  error  Strings must use doublequote                       quotes
  16:34  error  Strings must use doublequote                       quotes
  17:1   error  'Vue' is not defined                               no-undef
  17:15  error  Strings must use doublequote                       quotes
  17:39  error  Strings must use doublequote                       quotes
  20:1   error  'Vue' is not defined                               no-undef
  20:12  error  Strings must use doublequote                       quotes
  21:1   error  Expected indentation of 1 tab but found 4 spaces   indent
  22:1   error  Expected indentation of 2 tabs but found 8 spaces  indent
  22:16  error  'moment' is not defined                            no-undef
  22:45  error  Strings must use doublequote                       quotes
  23:1   error  Expected indentation of 1 tab but found 4 spaces   indent
  
  ....SNIP....
  
✖ 50 problems (50 errors, 0 warnings)
  40 errors, 0 warnings potentially fixable with the `--fix` option.

```

Running the same command with the `--fix` argument should get eslint to attempt to fix as many of the above errors as it can.

## Linting .vue files

Right now we can lint `.js` files but if we're using `.vue` single file components then trying to run eslint on them will give us an error like this:


```bash
./node_modules/.bin/eslint ./resources/assets/js/components/*.vue --fix                          

/resources/assets/js/components/Example.vue
  1:1  error  Parsing error: Unexpected token <
```

We need to tell eslint how to parse `.vue` files and what rules to apply to them.

```bash
# Yarn
yarn add -D eslint-plugin-vue

# NPM
npm install eslint-plugin-vue --save-dev
```

Next register the `eslint-vue-plugin` in your `.eslint.json` file by adding it to the extends block:

```javascript
    ...
    "extends": [
        "eslint:recommended",
        "plugin:vue/recommended"
    ],
    ...
```

There are actually a number of different default config values for the eslint vue plugin that add increasingly stricter rule sets to be applied to the linter. They are as follows:

```javascript
plugin:vue/base 
plugin:vue/essential
plugin:vue/strongly-recommended
plugin:vue/recommended
```

Now running eslint against `.vue` files will result in expected output:

```bash
/resources/assets/js/components/Example.vue
  20:3  error  Unexpected console statement  no-console
```

## Adding a new build step

Finally we can add eslint to our build process. For my work projects I use gitlab ci/cd. 

Start by adding a new `script` to our `package.json` file called `eslint` which we can call from gitlab:

```javascript
{      
  "script": {
    ...
    "eslint": "./node_modules/.bin/eslint resources/assets/js/ test/ --ext .js,.vue"
  },
}
```

This script tells eslint will look for all files with the `.js` and `.vue` extension recursively within the `resources/assets/js/` and `test` folders of my project.

Next lets update our `.gitlab-ci.yml` file to add the new build step. 

If you don't have a stage for `syntax` defined first do that:

```yaml
# Define pipeline stages
stages:
  - syntax
  - tests
  - deploy
```

Next add the job for eslint:

```yaml
eslint:
  image: node:6
  stage: syntax
  before_script:
    - yarn
  script:
    - yarn eslint
```

And that's it! Commit some code and watch your CI process check the code for errors!

## Finally

I found that I had a few rules defined by default that bugged me so i turned them off. My current `.eslint.json` file looks like this:

```javascript
{
    "env": {
        "browser": true,
        "commonjs": true,
        "es6": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:vue/recommended"
    ],
    "parserOptions": {
        "sourceType": "module"
    },
    "rules": {
        "indent": [
            "error",
            "tab"
        ],
        "linebreak-style": [
            "error",
            "unix"
        ],
        "quotes": [
            "error",
            "single"
        ],
        "semi": [
            "error",
            "always"
        ],

        "no-empty": 0,
        "no-unused-vars": 0,
        "no-undef": 0,
        "no-mixed-spaces-and-tabs": 0
    }
}
```


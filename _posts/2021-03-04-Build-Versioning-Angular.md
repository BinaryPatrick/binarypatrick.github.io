---
layout: post
title: 'Adding build versions in Angular'
permalink: 'angular-build-versioning'
date: 2021-03-04 19:30:00 -0500
edit: 2021-03-05 10:30:00 -0500
tags: ['Angular', 'Pipeline', 'DevOps']
---

Have you ever found yourself wondering which version of an SPA is running when you pull up your site? This was a problem I wanted to answer in my Angular SPA. Utilizing the `npm version` command in my build pipeline, I was able to include my pipeline build number in my app, significantly decreased troubleshooting time and version confusion.

<!--more-->

## Some considerations

Keep in mind, your build version numbers will need to be compliant with [normalized SEMVER2](https://github.com/NuGet/Home/wiki/SemVer-2.0.0-support#spec) to be valid for NPM.

## Instructions

### Step 1 - Adding node types to Angular

In your `tsconfig.app.json` file, add `node` to the `types` array. If there is not a already `types` array, add it under `compilerOptions` in the json root.

```json
{
  "compilerOptions": {
    "types": ["node"]
  }
}
```

### Step 2 - Adding environment variables

Find your `environment` constant in `src/environments`. You should see two files, one for prod, `environment.prod.ts`, and one for not-prod, `environment.ts`. In production, I keep it simple and add the `appVersion` variable without any changes.

```js
export const environment = {
  appVersion: require('../../package.json').version,
  production: true,
  base: 'area',
};
```

In the non-production environment though, I like to add a dev tag.

```js
appVersion: require('../../package.json').version + '--dev';
```

This helps me remember if I'm in a non production environment, because there is nothing worse than troubleshooting a problem in the wrong environment.

### Step 3 - Adding the version to an Angular component

Likewise this is easier than it sounds. Now that the node types and version variable are available, they just need to be accessed and displayed somewhere in the app. I prefer to add this to the bootstrapped component, which is usually `AppComponent`. This best way I've found to do this is with a host binding.

**Note**: _You do not need to specify `prod` vs `non-prod` in your environment import. When Angular builds, it will use the production environment if the `--prod` build flag is used._

```js
import { Component, HostBinding } from '@angular/core';
import { environment } from 'src/environments/environment';

@Component({
  ...
})
export class AppComponent {
  @HostBinding('attr.app-version') appVersionAttr = environment.appVersion;
}
```

### Step 5 - Adding `npm version` to your pipeline

If your pipeline stack offers automatically created version numbers, just make sure they are compliant with [normalized SEMVER2](https://github.com/NuGet/Home/wiki/SemVer-2.0.0-support#spec). Once you have add a task in your pipeline to run the npm command

```bash
npm version $(build.buildNumber)
```

The build variable above is for [Azure Devops](https://docs.microsoft.com/en-us/azure/devops/?view=azure-devops) specifically. Your pipeline stack may differ.

## Last Thoughts

Now that you've added a version number, build, build, build and check the check your work. You should see an html attribute on whatever html element you added the host binding to. For me it looks like this:

```html
<app-root app-version="3.1.28-master"></app-root>
```

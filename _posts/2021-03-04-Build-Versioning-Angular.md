---
layout: post
title: Adding build versions in Angular
---

Do you ever find yourself wondering which version of your SPA is running when you pull up your site? This was a problem I wanted to answer in my Angular SPA. Utilizing the `npm version` command in my build pipeline, I was able to include my pipeline build number in my app. This significantly decreased troubleshooting time and version confusion.

## Some considerations

Keep in mind, your build version numbers will need to follow SEMVER2 to be valid for NPM.

## Instructions

### Step 1 - Adding node types to Angular

In your `tsconfig.app.json` file, add `"node"` to the `types` array. If there is not a already `types` array, add it under `compilerOptions` in the json root.

### Step 2 - Adding environment variables

In production, I keep it simple. In the `environment` constant, I add the following line

```
appVersion: require('../../package.json').version
```

In the non-production environment, I like to add a dev tag.

```
appVersion: require('../../package.json').version + '--dev'
```

This helps me remember if I'm in a non production environment, because there is nothing worse than troubleshooting a problem in the wrong environment.

### Step 3 - Adding the version to your root Angular object

Likewise this is easier than it sounds. Now that the node types and version variable are available, they just needed to be added wherever you please in the app. I prefer to add this to the root app entry container. This can be done easily with a host binding.

```
export class AppComponent {
    @HostBinding('attr.app-version') appVersionAttr = environment.appVersion;
}
```

### Step 5 - Adding `npm version` to your pipeline

If your pipeline stack offers automatically created version numbers, just make sure they are SEMVER2 compliant. Once you have add a task in your pipeline to run the npm command

```
npm version $(build.buildNumber)
```

The build variable there is for Azure Devops. Your pipeline technology may differ.

## Last Thoughts

Now that you've added the version number, build baby build and check the check your work. You should see an html attribute on whatever element you added the host binding too. For me it looks like this

```
<app-root ng-version="7.2.16" app-version="3.1.28-master">
```

# Angular Configuration Service

This document describes the requirements and design decisions of `ng-config-service`: an Angular runtime configuration service.

## Introduction

There are two types of configuration in a software project: compile-time configuration and runtime configuraiton.

Compile-time configuration are settings used by a compiler to compile source code. A key differentiator of the compile-time configuraiton is that every change of the configuration requires a rebuild (or recompile) of all source code. Angular CLI comes with an extensible mechanism for the compile-time configuration. It is called `application enviornments` and is described in [an official Angular document](https://github.com/angular/angular-cli/wiki/stories-application-environments). Because it is used in build process, the configuration file can be a code file, as demonstrated by the Angular CLI that uses Javascript files to configure setttings for different build enviornments.

Runtime configuration is loaded at runtime to configure the behavior of an application. Runtime configuration file is usually a JSON file that is deployed with the application and is loaded when an application starts. Of course, there is no need of rebuild of source code when there is a configuration change. Some applications even allow detection of configuration chagne and dynamic reload of the configuration file at runtime.

Consequently, compile-time configuration is more appropriate for build process configuration while the runtime configuration is prefered for runtime configuration. Runtime configuration changes should not involve a rebuild of any source code.

Angular doesn't provide a standard mechanism to define and use runtime configuration. This document describes the requirments and design of runtime configuration for Angular applications.

## Requirments

The configuration data should be loaded during Angular bootstrap, before the application code runs.

The configuration data source can be a JSON file or an API that returns JSON data.

To make it simple, there are onlyh two public API: `load` to load configuraiton data and `get` to get a configuration value for a specific key.

To support AOT compilation, the configuration data path has to be provided during build time.

## Design

The runtime configuration implmentation has two parts: 1) a configuration service that loads configuration data and let others to get configuration settings, and 2) a provider initializes the configuration service during the Angular application boot process.

### Configuration Data URL

The configuration file is one kind of application asset. The location should be ignored by the compiler and is available in runtime. The `src/assets/config/config.json` is used as the default configuration data URL.

It's possbile to specify a configuration file path or a backend API URL to get the configuration data. The `InjectionToken` allows one to define a string value that can be opitonally injected to the configuration service.

### Configuration Service

The configuration service uses HTTP(S) protocol to get the configuration file and initialize the configuration settings. The configuration service should be a singlton created by the root injector。It has an `load` method that is called by a DI provider to load the configuration file. To make it simple, only one method, i.e., `get<T>(key: string): T | undefined` is defined to get configuration settings.

### Configuration Service Provider

The configuration service should be initialized during bootstrap process, before an app is initialized. The `APP_INITIALIZER` injection-token serves this puporse. It uses a factory that returns a promise. The factory function will provide the enviornment parameter and initialize the configuration service.

### Error Handling

If the configuration service fails to load the configuration data, due to either an invalid URL or wrong JSON syntax, it should stop the application and throw an excpetion. The exception message should give the failure reason.

How can an application continue with invalid configuration?

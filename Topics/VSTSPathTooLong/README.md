# Christian Gunderman's Blog - Visual Studio Team Services - VSTest Path Too Long

## Background

Last week, while putting together a Visual Studio Team Services Build Definition, I ran into an issue with a path too long error in the VSTest task that runs your Visual Studio Test Framework unit tests. I was stumped for a while because none of the paths were long than ~150 characters.

Version one of the task failed with something similar to 'Cmdlet Error, path too long', and version two failed with a node 'ENAMETOOLONG' failure.

## Problem

It turns out that the error message is actually incorrect and that the real problem is that I was exceeding the Powershell max *parameter count* because we were using the following VSTS-default minimatch pattern to enumerate the test assemblies:

`
**\*test*.dll
!**\obj\**
`

This pattern matches names like *FooTestHost.dll*, *FooTestHelper.dll*, and *FooTestFramework.dll*, of which, there were about 10 of each copied to each unit test project as copy-local dependencies, resulting in my file enumeration to grab more than 10 times the number of actual test assembly paths, exceeding the max parameter count.

## Solution

If you encounter this issue, fix it by adjusting your Visual Studio Team Services definition to use the following minimatch pattern instead, making note of the *s* in *tests*:

`
**\*tests*.dll
!**\obj\**
`

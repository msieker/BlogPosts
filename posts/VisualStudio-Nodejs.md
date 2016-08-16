Date: 2012-07-22 18:00
Title: Using Visual Studio for a node.js project
Category: Coding

I recently ran across the need to be able to work on some node.js code 
from within Visual Studio (The rest of the project is .Net, with node being
used for a few unique features). Previously I had done this work over a SSH
session to Emacs, but I decided I wanted something more integrated.

My first task was to make a project file that would work in Visual Studio,
with the build tasks being whatever I wanted to do to my Javascript, and
nothing else. To do that, I first chose to create a blank MVC project, then
delete all of it's contents and references. Then I opened up the project file
and removed the following line:

    :::XML
    <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />
    
And then added the following:

    ::XML
    <Target Name="Build">
    </Target>
    <Target Name="Clean">
    </Target>
    <Target Name="Rebuild" DependsOnTargets="Clean;Build">
    </Target>
    
Then the project was reloaded in Visual Studio to make sure all was well. I then
added my Javascript files to the project, ran a build, and make sure Visual Studio
didn't complain about anything.

Next up was the ability to run JSHint against the code in my project, while excluding
pre-minified files. I already had node.js on my path, so I installed jshint as a global
package with

    npm install jshint -g
    
After that, I grabbed the jshint reporter that worked with Visual Studio from the
[node-jshint-windows project](https://github.com/ArturDorochowicz/node-jshint-windows/blob/master/src/main/lib/vs_reporter.js)
and dropped it into my build libraries folder. Then I modified the `Build` task as follows:

    :::XML
    <Target Name="Build">
        <ItemGroup>
            <ToLint Include="@(Content)" Exclude="**\*.min.js" />
        </ItemGroup>
        <Exec Command="jshint --reporter &quot;$(SolutionDir)libraries\build\vs_reporter.js&quot; %(ToLint.Identity)" ContinueOnError="false" />
    </Target>
    
This takes any files within the `Content` ItemGroup created by Visual Studio, excludes
pre-minified files, and then runs jshint against them. The errors will then show up in the Error
Output window just like any other sort of build errors. If desired, `ContinueOnError` can be
changed to `true`, if you would rather JSHint errors be reported as warnings.
Title: Converting ASP.Net MVC2 project to MVC3
Slug: converting-asp-net-mvc2-project-to-mvc3
Date: 2010-11-21 19:55
Category: Coding

So, now that MVC3 has hit RC status, I wanted to upgrade a project I
was working on to take advantage of Razor. Here's what I've had to do so
far:

1.  Remove all System.Web.\* references (Except System.Web itself).
2.  Add in their place: System.Web.Mvc (Ensure it's the v3 assembly), System.Web.Webpages, System.Web.Razor, System.Web.Helpers.
3.  Go into each web.config in the project (At least one in the root of
    the project, and one in the root of of the Viewsfolder) and replace
    all instances of `System.Web.Mvc, Version=2.0.0.0,` with
    `System.Web.Mvc, Version=3.0.0.0,` in the various type definitions
    and assembly blocks.
4.  In the root `web.config` add these lines to the `appSettings` section:

        :::XML
        <add key="ClientValidationEnabled" value="true"/>
        <add key="UnobtrusiveJavaScriptEnabled" value="true"/>

    Also ensure all of the following are in the `system.web/pages/namespaces` element:

        :::XML
        <add namespace="System.Web.Helpers" />
        <add namespace="System.Web.Mvc" />
        <add namespace="System.Web.Mvc.Ajax" />
        <add namespace="System.Web.Mvc.Html" />
        <add namespace="System.Web.Routing" />
        <add namespace="System.Web.WebPages"/>

    And that these are within `system.web/compilation/assemblies`:

        :::XML
        <add assembly="System.Web.Abstractions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
        <add assembly="System.Web.Helpers, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
        <add assembly="System.Web.Routing, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
        <add assembly="System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
        <add assembly="System.Web.WebPages, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />

5. In the `web.config` in the root of Views, add this:

        :::XML
        <configSections>
            <sectionGroup name="system.web.webPages.razor" type="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35">
                <section name="host" type="System.Web.WebPages.Razor.Configuration.HostSection, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
                <section name="pages" type="System.Web.WebPages.Razor.Configuration.RazorPagesSection, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
            </sectionGroup>
        </configSections>

        <system.web.webPages.razor>
            <host factoryType="System.Web.Mvc.MvcWebRazorHostFactory, System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
            <pages pageBaseType="System.Web.Mvc.WebViewPage">
                <namespaces>
                    <add namespace="System.Web.Mvc" />
                    <add namespace="System.Web.Mvc.Ajax" />
                    <add namespace="System.Web.Mvc.Html" />
                    <add namespace="System.Web.Routing" />
                </namespaces>
            </pages>
        </system.web.webPages.razor>

    If you already have a `<configsections>` element, add the `<sectiongroup>` block to that.

6.  Right click on the project in Solution Explorer and select "Unload
    Project". Right click on it again, and select "Edit [Project Name]".
    Find the <ProjectTypeGuids> tag and replace it with:

        :::XML
        <ProjectTypeGuids>{E53F8FEA-EAE0-44A6-8774-FFD645390401};{349c5851-65df-11da-9384-00065b846f21};{fae04ec0-301f-11d3-bf4b-00c04f79efbc}</ProjectTypeGuids>

    Once, done, save, close the file, right click on the project in
    Solution Explorer and select "Reload". This grants you the fun new
    Add View dialogs and such.

And that should do it.

# Sample applications for the Microsoft Teams developer platform

These code samples show you the capabilities of Microsoft Teams apps and various ways to implement those capabilities:

* **[Get Started Sample](https://github.com/OfficeDev/microsoft-teams-sample-get-started)**&emsp;This sample shows all the capabilities available in a Microsoft Teams app, including bots, tabs, compose extensions, and connectors. Source code is provided in both C# and Node.js.
* **[Microsoft Graph API Samples](https://github.com/OfficeDev/microsoft-teams-sample-graph)**&emsp;These samples demonstrate using Microsoft Graph API calls to perform tasks such as querying teams and channels from a web service running outside Microsoft Teams.
* **["To-do" list sample tab app](https://github.com/OfficeDev/microsoft-teams-sample-todo)**&emsp;This Node.js sample shows how easy it is to convert an existing web app into a tab.

## Common prerequisites

We recommend the following common prequisites for running our sample experiences:

* [An Office 365 account with access to Microsoft Teams, with sideloading enabled](setup.md)
* For Node.js:
    * [Visual Studio Code](https://code.visualstudio.com/)
    * [Node.js](https://nodejs.org/en/download/)
* For .NET/C#:
    * Visual Studio (you can download the [Community](https://www.visualstudio.com/free-developer-offers/) version for free)
* For samples with bots: [Bot Framework Emulator](https://docs.microsoft.com/en-us/bot-framework/debug-bots-emulator)
* [Git command line tool](https://git-scm.com/downloads) or [Git for Windows](https://git-for-windows.github.io/)
* Tunneling software like [ngrok](https://ngrok.com/download)

## Getting samples

Microsoft hosts much of its sample code in GitHub, a web-based Git repository hosting service. If you’re not familiar with Git or GitHub, we recommend that you review the [Git documentation](https://git-scm.com/doc) and follow the [Hello World](https://guides.github.com/activities/hello-world/) project in GitHub Guides.

To download our samples from GitHub:

1. Open the project in GitHub
2. Choose the **Clone or download** button and copy the URL
3. Open a command prompt in the parent directory into which you want to install the sample project
4. Run `git clone <pasted url>`

This process copies the entire project to your computer. Most samples containing bots provide code in both Node.js and .NET/C#; you can find the code you want in the appropriate subdirectory.

### For Node.js samples

We provide a packages.json file that lists all required packages for a sample. Simply run `npm install` from the command line in your Node.js project directory to install the required packages. You're now ready to open the project in Visual Studio Code and start experimenting.

### For .NET/C# samples

Each of our .NET samples includes a Visual Studio solution file that contains all required libraries.

### For other samples

As always, the project's README file should have more information on specific needs for specific samples.

## Tips for running Microsoft Teams samples

Our sample apps attempt to standardize much of the configuration and structure to simplify your experience. Although each project might have specific additional instructions within its README file, a few commonalities are discussed here.

### Project structure

For Microsoft Teams app projects, we provide a representative app package (.zip) along with the granular components (icons and manifest.json) in a subfolder named "TeamsAppPackage". In general, these packages are for your convenience in building and sideloading the sample app; they won't run without modification. Please review the manifest file for guidance on which values should be replaced to repackage and sideload your sample for running within Microsoft Teams.

#### Node.js projects

For Node.js projects, we provide a Visual Studio Code configuration file (launch.json), which contains profiles for running in the Bot Emulator (for projects with bots) and for running within Teams. These profiles set default environment variables for you to edit with your own values, such as the following:

* `MICROSOFT_APP_ID`&emsp;Set to your registered Bot Framework bot ID (or blank for the Emulator configuration)
* `MICROSOFT_APP_PASSWORD`&emsp;Set to your registered Bot Framework bot ID (or blank for the Emulator configuration)
* `TEAMS_APP_ID`&emsp;Set to your registered Bot Framework bot ID, or different as needed
* `BASE_URI`&emsp;The base URL for your local hosted instance (such as in tab sample projects); most likely set to your tunnel URL from ngrok or other tunneling software for local debugging, or to your production endpoint if cloud-hosted

#### .NET/C# projects

For .NET/C# projects, we leverage the web.config file to host environment variables for our project, which you should modify to run as needed for your testing. We use the following environment variables, which are similar to those listed in the previous section:

* `MicrosoftAppId`
* `MicrosoftAppPassword`
* `TeamsAppId`
* `BaseUri` 

### Running and debugging

For general information on running and debugging our samples, see [Run and debug your Microsoft Teams app](debugging.md).

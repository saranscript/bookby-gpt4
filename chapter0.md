Chapter 0: Setting up the Development Environment with VSCode for Back-end, Front-end, and Database

Introduction:
Before diving into the development of the Thee.ai social media app, it is crucial to set up a development environment that will allow you to work efficiently with the necessary tools and technologies. In this chapter, we will walk you through setting up Visual Studio Code (VSCode) as your integrated development environment (IDE), configuring it for back-end (Django and Django REST Framework), front-end (React and TypeScript), and database (PostgreSQL) development.

0.1.1 Installing Visual Studio Code

Visual Studio Code, or VSCode, is a popular, lightweight, and powerful code editor developed by Microsoft. It is free and open-source, and it is available for Windows, macOS, and Linux. To install VSCode, follow the instructions on the official website: https://code.visualstudio.com/download

0.1.2 Installing and Configuring Python

To work with Django and Django REST Framework, you need to have Python installed on your system. Follow the instructions on the official Python website to install the latest version of Python: https://www.python.org/downloads/

Once Python is installed, you should also set up a virtual environment to isolate the dependencies of your project. To do this, open a terminal or command prompt, navigate to your project folder, and run the following commands:

python -m venv venv

To activate the virtual environment, run the appropriate command for your system:

Windows: venv\Scripts\activate
macOS/Linux: source venv/bin/activate

0.1.3 Installing and Configuring Node.js and npm

To work with React and TypeScript, you need to have Node.js and npm (Node Package Manager) installed on your system. Follow the instructions on the official Node.js website to install the latest version of Node.js and npm: https://nodejs.org/en/download/

0.1.4 Setting up PostgreSQL

To work with PostgreSQL, you need to have it installed on your system. Follow the instructions on the official PostgreSQL website to install the latest version of PostgreSQL: https://www.postgresql.org/download/

Once PostgreSQL is installed, create a new database and user for your Thee.ai project. You can use the command-line tool psql or a graphical user interface like pgAdmin to do this.

0.1.5 Configuring VSCode Extensions

VSCode has a rich ecosystem of extensions that can help you streamline your development process. Here are some recommended extensions for our Thee.ai project:

Python (Microsoft): Provides Python language support, debugging, linting, and code formatting.
Pylance (Microsoft): Provides advanced Python language support, including type checking and auto-import suggestions.
Django (Baptiste Darthenay): Adds Django-specific syntax highlighting and code snippets.
Django Template (bibhasdn): Enhances Django template support in VSCode.
ES7 React/Redux/GraphQL/React-Native snippets (dsznajder): Provides code snippets for React, Redux, GraphQL, and React Native.
TypeScript (Microsoft): Provides TypeScript language support, including syntax highlighting, code navigation, and refactoring.
PostgreSQL (Chris Kolkman): Adds PostgreSQL support to VSCode, including syntax highlighting, linting, and query execution.

To install an extension, open the Extensions view in VSCode by clicking on the Extensions icon in the Activity Bar on the side of the window or by pressing Ctrl+Shift+X. Search for the desired extension, and click the Install button.

0.1.6 Configuring Project Settings

In addition to installing the necessary extensions, you should also configure some project settings in VSCode. To do this, create a vscodefolder in your project root directory and add asettings.jsonfile inside it. This file will store your project-specific settings. You can also configure global settings by opening the settings editor in VSCode withCtrl+,` or by navigating to File > Preferences > Settings.

Here's a sample settings.json file for our Thee.ai project:

{
  "python.pythonPath": "venv/bin/python",
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": false,
  "python.linting.flake8Enabled": true,
  "python.formatting.provider": "black",
  "editor.formatOnSave": true,
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.preferences.quoteStyle": "single",
  "typescript.preferences.importModuleSpecifier": "relative",
  "javascript.preferences.quoteStyle": "single",
  "javascript.preferences.importModuleSpecifier": "relative",
  "files.associations": {
    "**/*.html": "django-html",
    "**/templates/**/*.html": "django-html",
    "**/templates/**/*": "django-txt",
    "**/requirements{/**,*}.{txt,in}": "pip-requirements"
  }
}


This configuration:

Sets the Python interpreter to the one in the virtual environment.
Enables linting with Flake8 and disables Pylint.
Sets the formatting provider to Black, a popular code formatter for Python.
Formats code on save.
Configures TypeScript settings to use single quotes and relative imports.
Associates Django-specific file types with the appropriate language modes for syntax highlighting and IntelliSense.
0.1.7 Finalizing the Development Environment

Now that you have installed and configured the necessary tools and extensions, your development environment is ready for back-end, front-end, and database development.

In summary, you have:

Installed Visual Studio Code.
Installed and configured Python, Node.js, and PostgreSQL.
Set up a Python virtual environment.
Installed and configured VSCode extensions for Python, Django, React, TypeScript, and PostgreSQL.
Configured project-specific settings in VSCode.
With your development environment ready, you can now proceed to the next chapter, where we will begin developing the Thee.ai social media app using Django, Django REST Framework, React, TypeScript, and PostgreSQL.




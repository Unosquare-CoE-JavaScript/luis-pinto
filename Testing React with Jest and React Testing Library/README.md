## React Testing Library 
- Not just a library , also a philosophy
- Test your software the way user actually use it,not internal implementation
- Find elements by accessibility markers, by the way screen readers and othe assistive technologies
## react Testing Library vs Jest
### React Testing Library 
Provides virtual DOM for test
### Jest
Test Runner ,responsable for:
- Find test
- Run Test
- Determines whether test pass or fail
## How does jest Work
Global *test* method has 2 arguments:
- String description
- Test function

Test fail if error is thrown when running function
Assertions throw errors when expectation fails
## TDD
Write test before writing code

Then write code according to "spec" set by tests

"Red-greend" testing

Test fail before code is written
## Type of Tests
- Unit test 
Test one unit of code in solation
- Integration test
Test how multiple units work together
- Functional test
Test a particular function of a software
- Acceptace/End-to-end Test
Use actual browser and server (Cypress, Selenium)

## Functional Testing
Differense mindset from unit testing
### Unit testing
Isolated: Mock dependencies , test internals

:star: Very easy to pinpoint failures

:thumbsdown: Further from how users interact with software

:thumbsdown: More likely to break with refactoring

### Functional testing
Include all relevant units, test behavios

:star:Close to how users interact with software

:star:Robust tests

:thumbsdown: More difficult to debug failing tests

## TDD vs BDD
Testing library encourage testing *behavior* over *implementation*

BDD is very explicity defined

Involves collaboration between lots of roles

- Developers , QA, Business partners

Defines process for different groups to interact
## Accessibility and finding Elements
Testing Library recommends finding elements by accessibility handles

https://testing-library.com/docs/queries/about/#priority
## Review: Setting up ESLint and Prettier
ESLint:
- *npm install eslint-plugin-resting-library eslint-plugin-jest-dom*
- Remove *eslintConfig* from package.json
- Create *.eslintrc.json* and add standard config
- Install ESLint extension for VSCode
- Create *.vscode/settings.json* and add standard config
- Add *.eslintcache* and *.vscode* to .gitignore

Prettier:
- Add settings to .vscode/settings.json
- Install Prettier on VSCode
## await findBy
**When you are waiting for something to appear asynchronously on the page you must use *await findBy***
## Review pf "Scoops" Testing
Mocke Service Worker mimics response from the server
- Create handler
- create server
- update setupTests to listen for requests

*getAllByRole*
- Search for more than one match to role

*await* ***find**AllByRole*
- for asynchronous DOM update for elements

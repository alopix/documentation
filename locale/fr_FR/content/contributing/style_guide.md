- - -
Title: Style Guide Date: 2021-10-20 Order: 4
- - -

## Pull requests

Alors, vous voulez contribuer au code de BookWyrm, c'est génial! If there's an open issue that you'd like to fix, it's helpful to comment on the issue so work doesn't get duplicated. Try to keep the scope of pull requests small and focused on a single topic. That way it's easier to review, and if one part needs changes, it won't hold up the other parts.

Si vous ne savez pas comment régler un problème, ou que vous n'êtes plus disponible pour le faire, ne vous en faites pas. Laissez simplement un commentaire sur la requête et nous allons prendre le relais 💖.

Pull requests have to pass all the automated checks before they can be merged - this includes style checks, global linters, a security check, and unit tests.

## Linting

### Global

We use [EditorConfig](https://editorconfig.org) to maintain consistent indenting and line endings.

### Python

BookWyrm uses the [Black](https://github.com/psf/black) code formatter to keep the Python codebase consistent styled. All new pull requests are checked with GitHub actions, and you can automatically fix code style problems by running `./bw-dev black`

Code is also checked with Pylint using GitHub Actions. Pylint warnings must be addressed before pull requests are merged, but it's a judgement call if the suggestion should be used, or the warning suppressed. To suppress a warning, add a comment at the end of or on the line above the warnings: `# pylint: disable=warning-name`

### Templates (HTML)

Your pull request will also be checked by the [curlylint](https://www.curlylint.org) linter for Django templates.

### CSS

We use [stylelint](https://stylelint.io) to check all CSS rules. As with Pylint [you can disable stylelint](https://stylelint.io/user-guide/ignore-code) for a particular rule, but you will need a good justification for doing so.

### JavaScript

[ESLint](https://eslint.org) checks any JavaScript changes you have made. If ESLint doesn't like your working JavaScript, check the linter message for the exact problem.

## Design inclusif

BookWyrm aspire à être aussi inclusif et accessible que possible.

When contributing code, check the [Inclusive Web Design Checklist](https://github.com/bookwyrm-social/bookwyrm/discussions/1354) before you file your pull request. For accessibility advice, [A11Y-101](https://www.a11y-101.com/development) is also a useful source. For information on how to make your page templates multi-lingual, see the [Translations section](/translations.html).

Some particular things that Bookwyrm contributors have found useful to remember are:

### Forms

* Only use `input[type="checkbox"]` or `input[type="radio"]` inside `<label>`
* If you do not place checkboxes and radio buttons inside the `<label>`, the `<label>` should be placed _after_ the element it relates to

### Buttons and Links

* Use a `<button>` element for anything that exists to trigger a JavaScript action (e.g. hiding or unhiding a form) or sends a `POST` request (e.g. submitting a form)
* Use an `<a>` element for anything that triggers a `GET` request. Usually, an anchor (`<a>`) element should not be styled as a button (`class="button"`), though there are some exceptions, such as "Cancel" buttons. If in doubt, ask for advice in your pull request

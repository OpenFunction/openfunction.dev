# OpenFunction Website

This project uses [Hugo](https://gohugo.io/) and [Hugo template Docsy](https://github.com/google/docsy) to build the OpenFunction website.

## Contribute

Contributions of any kind are welcome!

### Fork and clone the repo

First, create your own fork of the repository.

Then, clone your fork and enter into it:

```
git clone https://github.com/OpenFunction/website.git
cd website
```

### Compiling and preview the website

You will need to install [Hugo](https://gohugo.io/) to build the website in order to **publish it as static content.**

#### Install Hugo extended

Go to the [Hugo releases place](https://github.com/gohugoio/hugo/releases) and download the `hugo_extended` version that better suits your OS.

**EXTENDED version is MANDATORY to properly build the static content!**

Note: If you install Hugo on Windows, you need to add environment variables for the exe file of Hugo. Execute `hugo version` to view if the installation is successful.

### Running the website locally

When you have installed Hugo, then run:

```
hugo server
```

Now you can preview the website in your browser using `http://localhost:1313/`.

### Open a pull request

Open a [pull request (PR)](https://help.github.com/en/desktop/contributing-to-projects/creating-an-issue-or-pull-request#creating-a-new-pull-request) to contribute to our website. Please use DCO sign-off when you submit a pr. Refer to the command below (add `-s`):

```bash
git commit -s -m "xxx"
```

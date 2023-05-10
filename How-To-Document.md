# How to Document

Documentation is important. Especially when developing a complex application in a growing team, good documentation helps to capture and share knowledge transparently. Examples are guides on how to complete certain tasks, install and use required software, coding guidelines, and architecture decisions.

Meetings and the verbal sharing of information can be a start, but eventually, a team needs an always accessible source of information. It reduces misunderstandings and a lot of redundant communication - it's far better to spend time on good documentation than repeatedly having to answer the same questions.

Documentation can also help further your understanding. Once you write about something in a structured way, you'll notice if there are gaps in your knowledge.

Since documentation must evolve with the code base, simple and flexible tooling is required to keep everything up to date.
In this document, I share a simple way to write well-formatted, structured, and sharable documentation. I also show how to include diagrams in a versatile manner.

## Formats

### Markdown

Markdown is an easy-to-write, well-supported format. It is also supported by GitHub. I recommend it for simple documentation that fits into a single file.

To write Markdown, use tools like [Typora](https://typora.io/) or your favorite code editor. [VS Code](https://code.visualstudio.com/), for example, comes with a Markdown preview.

### Asciidoc

For complex documentation that requires a table of contents, [Asciidoc](https://asciidoc.org/) provides more flexibility. It is also supported by GitHub.

If you feel more comfortable writing Markdown, you can start with it, and if the need arises, [convert to Asciidoc](https://github.com/asciidoctor/kramdown-asciidoc) later.

To write Asciidoc, you can install the Asciidoc extension by [Asciidoctor](https://asciidoctor.org/) for VS Code. It also provides a preview.

### Diagrams.net

If diagrams are required, generate those with the offline version of [diagrams.net](https://www.diagrams.net/). Compared to PlantUML, it offers better ways to visualize complex diagrams. Those can get chaotic quickly with PlantUML.

For proper interoperability between diagrams.net and Markdown / Asciidoc, the diagrams should be saved as SVGs. Such SVGs include both the code used by diagrams.net and the SVG. It allows for the embedding of those diagrams:

```
# Markdown
![](relative link to svg)

# Asciidoc
image::relative-link-to-svg.svg[Static]
```

You can also re-open the SVGs in diagrams.net and make further adjustments.

## Structure

A common understanding of where different types of documentation are located is essential. I suggest the following structure:

- **base folder** of the project - **README.md** with developer documentation:

  - General information about the project
  - How to install dependencies
  - How to develop
  - How to compile
  - How to deploy
  - ...

- **doc** sub-folder:

  - **architecture.md** or **architecture.adoc** file - Architecture decisions, used technologies, and resources should be documented here

  - **doc/diagrams** sub-folder - If diagrams are required, those should be saved as SVGs

If you use a Monorepo, containing several services/apps, those might require specific documentation. You can again place a README with developer documentation in the different *base folder*s. All other documentation should reside in the top-level *doc* folder though. It makes it easier, to export the documentation to other formats.

## Export

We already covered some editor options for Markdown and Asciidoc above. Those will have you covered to distribute your documentation via Github. For export to other formats, additional tools are required.

### Markdown - Pandoc

If your documentation uses Markdown, use [Pandoc](https://pandoc.org/) to convert it to html and PDF. Under Ubuntu, install the following tools:

- Pandoc - [Installation instructions](https://pandoc.org/installing.html#linux) for Ubuntu - for simplicity, download the gz file and unpack to ~/.local as described in the instructions

- Latex

  ```
  sudo apt install texlive-latex-base
  sudo apt install texlive-latex-recommended
  sudo apt install texlive-latex-extra
  ```

- rsvg converter
  ````
  sudo apt install librsvg2-bin
  ````

As an example, the following command will convert the architecture.md file to a PDF:

````
pandoc architecture.md -f markdown -t pdf -s -o architecture.pdf
````

### Asciidoc - Asciidoctor

If you use Asciidoc and want to convert to html and PDF, [Asciidoctor](https://asciidoctor.org/) shall be used.

- To create html files, install aciidoctor:
  ````
  sudo apt install -y asciidoctor
  ````

- To create PDF files, install asciidoctor-pdf:
  ````
  # ruby is required 
  sudo apt install ruby
  
  # install asciidoctor-pdf
  sudo gem install asciidoctor-pdf --pre
  ````

  More information about additional plugins, you might require can be found [here](https://comtronic.com.au/how-to-install-and-configure-asciidoctor-pdf/).

As an example, the following command will convert the architecture.adoc file to a PDF:

````
asciidoctor-pdf architecture.adoc
````

## Linting

If you're writing documentation, you can use linting to ensure proper spelling and grammar. [LanguageTool](https://github.com/languagetool-org/languagetool) is a good, open-source choice for that.

For Visual Studio, you can install the [LanguageTool Extension](https://github.com/davidlday/vscode-languagetool-linter). Read the instructions in the Readme, to configure it properly. Out of the box, it will support Markdown already. To also support Asciidoc, head to the extension's settings and add "asciidoc" to the *Language Tool Linter > Plain Text: Language Ids* setting.



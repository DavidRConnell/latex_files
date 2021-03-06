# LaTeX Documentation

The goal of this program is to remove the redundancies in producing LaTeX
documents and to integrate MATLAB and python programs into LaTeX projects
easing the embedding of high quality plots and tables.

LaTeX allows for a lot of flexibility setting document style and
formatting however, by default these settings must be rewritten for each
document as well as elements within documents, like writing float
environments for every float when most of the environment block is the same
for each figure or table.
In addition to reducing typing and clutter, formatting settings are kept in
a central location so changes only need to be made once instead of for
every instance.
This package generates tex files based on an rc file, leaving you to worry
only about the documents content.
Additionally, any template class can be used and, with some LaTeX
background, new classes can be created from those provided.

## Installation

This program requires a unix based file hierarchy, bash, and LaTeX.
To install, cd into the parent directory you want to use to store the
program and run:

	git clone https://github.com/DavidRConnell/latex_documentation.git
	./install.sh

This links the shell scripts and LaTeX packages and classes to the appropriate
locations and generates a docsrc resource file in ~/.config.

To install the additional MATLAB and python files for creating tables and
figures run the following commands:

	./installmatlabfiles.sh /path/to/matlab/dir
	./installpythonfiles.sh /path/to/python/dir

Where the paths should be replaced with the desired locations which need to
be in the MATLAB/python path or added to it.
For MATLAB, if you don't know where to link the files, ~/Documents/MATLAB
will likely be a good choice.
Otherwise running the following commands in the MATLAB command line will
add a new directory to MATLAB's path:

	addpath /path/to/dir
	savepath

Python's path is saved as an environment variable named PYTHONPATH.
To check the current path open a terminal and run:

	echo $PYTHONPATH

It may not exist yet.
To add a new directory run:

	export PYTHONPATH='path/to/dir':$PYTHONPATH

This will not be saved once the session ends; add the above line to
~/.bashrc or the analogous file for the shell you use to make the change
persistent.

**Note:** The [matlab2tikz](https://www.mathworks.com/matlabcentral/fileexchange/22022-matlab2tikz-matlab2tikz) package is needed for generating tikz figures in MATLAB and the
[matplotlab2tikz](https://pypi.org/project/matplotlib2tikz/) is needed for
python along with the matplotlib and numpy modules.

## Set up

The docsrc file created in ~/.config sets global documentation options.
Additionally, local docsrc files can be made to override the global file for
projects below the file in the directory hierarchy.

An example docsrc file should look like:

	AUTHOR="John Doe"
	EMAIL="john_doe@aol.com"
	DOCTYPE=rushdoc

	MAINFONT=helvetica
	TITLEFONT=helvetica

	NUMBERPAGES=false
	USETWOCOLUMN=true
	ADDABSTRACT=true

	DOEXPORTPDF=true
	DOCPATH="~/documents/documentation:~/documents/presentations"
	EXPORTPATH="~/Dropbox/documentation:~/Dropbox/presentations"

Local docsrcs can be added to the parent directory of a project or into
directory in `DOCPATH` and will override the values for all projects in
that directory.
Only the variables to change need be set i.e. to change the author and email
for one directory create a docsrc containing:

	AUTHOR="Jane Smith"
	EMAIL="jane_smith@netscape.com"

The first five values may be left blank, resulting in empty values in the
main file (that can be manually added later).
The `mkdoc` command will still generate a main file with an unset doctype but
`latexmk` will fail unless the class is set manually.

`NUMBERPAGES` to `DOEXPORTPDF` should be either true or false (bash treats an
unset vale as true).
The remaining two options can safely be ignored when `DOEXPORTPDF` is false.
(Although, `DOCPATH` affects where docsrc are looked for so there may still
be a reason to set it.)

If `ADDABSTRACT` is true `\maketitleandabstract{filename}` is used in place
off `\maketitle` where filename is the file containing the abstract located
under the sections sub directory (similar to adding sections in [adding
content](#addcontent)).

All values consisting of more than one word should be wrapped in "" (and
all values can be wrapped in "" if desired).

`DOCTYPE` should be the name of a template class.
Example template classes rushdoc and rushpresentation are contained in the
classes directory (see [Creating additional classes](#classes) for more info).
The term "template class" here simply means a cls file that sets style
and formatting.

The fonts can be any font installed on your machine.
Title font controls the font of the section headings as well as the title.

### Exporting the PDF

To export the new PDF to another directory (possibly a shared directory
where you don't want the source code making a mess) on creation set
`DOEXPORTPDF` to true and add the directory name where all the projects
which are to be exported will be stored under to `DOCPATH` and the directory
where the PDFs are to be copied into in `EXPORTPATH`.
This can be a multilevel hierarchy.
For example `DOCPATH` may contain subdirectories for several groups of
documentation.
For every project in one of these subdirectories, whenever the PDF
is created with `buildtexpdf` it will be copied to the appropriate
`EXPORTPATH` with the same file tree.
As an example if you are exporting the file in
`~/Documents/papers/example` to the empty export path `~/Dropbox/Documents`
and `DOCPATH` is set to `~/Documents` when the PDF is built it will be
copied to `~/Dropbox/Documents/papers/example.pdf` creating the papers
directory in the process.

If documents are going to be stored in multiple directory hierarchies
additional paths can be appended to `DOCPATH` and `EXPORTPATH` separated by a
":".
There are a couple ways to tell `buildtexpdf` where to copy the built PDF.
If `EXPORTPATH` is a single path the file is sent to that path.
This means the `EXPORTPATH` can be set in a local docsrc instead of the
global docsrc.
Alternatively, all the paths can be set globally, in which case `EXPORTPATH`
is expected to be in the same order as `DOCPATH` as in the example file
above.
In case there's uncertainty in if/where the PDF will be sent (or was sent
if it didn't end up where it was supposed to), the dry run option can be
passed to `buildtexpdf --dry-run`.
This will print information about exporting the file without creating it.

## Use

### New projects

To create a new project, after setting up ~/.config/docsrc, run:

	mkdoc projectname title

With appropriate values for `projectname` and `title`. This makes a new
directory with the given name populated with a sections directory and
`projectname`.tex (what I refer to as the main file) generated using
`title` and the values from docsrc.

From here on out, usage will be explain by creating the project "example".
To create the project in ~/documentation directory:

	cd ~/documentation
	mkdoc example "LaTeX Documentation Example"
	cd example

At this time the file tree looks like:

	example
	├── example.tex
	└── sections/

Running `buildtexpdf example` should produce example.pdf.

**Note:** Adding a docsrc to example would not make any changes.
For settings that affect all (and only) projects within ~/documentation a
docsrc should be added to ~/documentation/docsrc.

### <a name="addcontent"></a>Adding content

Sections added to the sections directory can be input to the project by
adding to the main file:

	...
	\begin{document}
		\pagenumbering{gobble}
		\maketitle

		\inputsection[optional name]{filename}
	\end{document}

If no optional name is included, the section will be named as the
capitalized file name otherwise the optional name is used.
In addition to the plain `\inputsection` macro, a star version (i.e.
`\inputsection*{filename}`) exist which does not add a section header.

**Note:**: `inputsection` searches the sections directory by default so there's
no need to prepend sections to the filename nor do you need to add the tex
extension.

Similarly, there are `inputsubsection` and `inputappendix` macros.
Subsection should be kept in the sections directory and appendices in an
appendices directory.
The appendix switch is needed before inputing any appendices to tell LaTeX
to start numbering the sections as appendices.
In the main file add:

	...
	\begin{document}
	...
		\appendix
		\inputappendix[optional name]{filename}
	\end{document}

### Generating floats

If the MATLAB and/or python files have been installed plots can be saved as
tikz figures, and matrices and numpy arrays can be saved in LaTeX table
format.
These files are saved to the figures and tables subdirectories respectively
within the provided LaTeX project.

Using MATLAB a sin wave plot and random data can be saved to the example
project with:

generateFloats.m

	function generateFloats
		time = 1:0.1:10;
		wave = sin(2* pi * time * 0.25);

		plot(time, wave)
		xlabel('time')
		ylabel('sin')
		saveFigForLatex('matlabfigure', '~/documentation/example');

		d.cheader = {'names', 'w', 'x', 'y', 'z'};
		d.rheader = {'a', 'b', 'c', 'd', 'e'};
		d.data = rand(5, 4);

		saveTableForLatex(d, 'matlabtable', '../test/this/');
	end

This will also create the figures and tables subdirectories if they don't
yet exist.
Now the file tree will look like:

	example
	├── example.tex
	├── figures
	│   └── matlabfigure.tex
	├── sections/
	└── tables
		└── matlabtable.tex

If the same parent directory for documentation is going to be used often it
may be worth creating a wrapper for `saveFigForLatex.m` and
`saveTabForLatex.m` that includes the path to documentation.
For example:

saveFigForDocumentation.m

	function saveFigForDocumentation(name, project)
		% Wrapper for saveFigForLatex to simplify saving to the documentation
		% directory.
		%
		% USAGE: saveFigForDocumentation(figureName, project);

		projectPath = ['~/documentation/', project];
		saveFigForLatex(name, projectPath);
	end

An analogous program for python would be:

generate_floats.py

	import save_for_latex as sfl
	import matplotlib.pyplot as plt
	import numpy as np

	sfl.set_project_path('~/documentation/example')

	t = np.linspace(1, 10, 100)
	s = np.sin(2 * np.pi * t * 0.25)

	plt.plot(t, s)
	plt.xlabel('time')
	plt.ylabel('sin')
	sfl.fig('pythonfigure')

	data = np.random.rand(3, 4)
	rows = ['a', 'b', 'c']
	columns = ['names', 'w', 'x', 'y', 'z']
	sfl.table('pythontable', data, rows, columns)

Including floats in the LaTeX document is done with the `inputfigure`,
`inputtable`, and `inputsubfigure` macros of the form:

	\inputfigure[\label{fig:ex}Caption]{figurename}
	\inputtable[\label{tab:ex}Other caption]{tablename}

	\begin{figure}[ht]
		\centering
		\inputsubfigure[%
			\label{fig:subfiga}caption for subfig a
		]{figurewidth}{subfiga}
		\inputsubfigure*[%
			caption for subfig b
		]{figurewidth}{subfigb}
		\inputsubfigure[%
			caption for subfig c
		]{figurewidth}{subfigc}
		\caption{\label{fig:subfigures}%
			General caption.
		}
	\end{figure}

Tables are expected to be in ./tables and figures in ./figures.
File extensions are not needed.
If there are multiple figures with the same name but different extensions
the extension with the highest priority will be chosen starting with .pgf
then .tex after that LaTeX makes the decision based on its own criteria.

The labels and captions are not required.
If labels are used `\autoref{fig:ex}` can be used to reference them in the
text adding the type based on whats before the ":" (see
[referencing](https://en.wikibooks.org/wiki/LaTeX/Labels_and_Cross-referencing)
for more info).

The star variant of `\inputsubfigure` places the next subfigure on the line
below.
In the example above the first two subfigures are on the same line and the
third is underneath them.

Adding the floats generated above is down by creating a new section:

	vi section/addingfloats.tex

Then adding the following:

section/addingfloats.tex

	Here we try adding some floats to a document.

	\inputfigure[%
		\label{fig:mat} Heres the plot produced with generateFloats.m
	]{matlabfigure}

	Now for some subfigures.
	\begin{figure}[ht]
		\centering
		\inputsubfigure[%
			\label{fig:py} Subfigures can be labeled separately
		]{0.45\textwidth}{pythonfigure}
		\inputsubfigure*[Another subfigure]{0.45\textwidth}{matlabfigure}
		\inputsubfigure[The last subfigure]{0.45\textwidth}{pythonfigure}
		\caption{\label{fig:sub}The entire subfigure cluster can also be
		referenced as a whole. And \autoref{fig:py} can be refernced in here.}
	\end{figure}

	Down here we can reference the top MATLAB figure with \autoref{fig:mat} and
	one of the python figures with \autoref{fig:py}

	Additionally a table can be printed with:

	\inputtable[The table created with generate\_floats.py]{pythontable}

Then the add the new section to the main file:

example.tex

	...
	\begin{document}
		\pagenumbering{gobble}
		\maketitle

		\inputsection[Add Floats]{addingfloats}
	\end{document}

Now example.pdf can be generated:

	buildtexpdf example

## <a name="classes"></a>Creating Additional Classes and Packages

Creating classes gives you control over document style and formatting.
Any class which uses the base.sty package (included as part of this program),
directly or through another package which calls it, can be used as a doctype.
The base.sty package sets up the fonts, colors, and calls other LaTeX
packages that I always want when writing papers.
All the macros needed for using this project are available after base.sty
is called.

The packages paperformat.sty and presentation.sty extend base.sty to
include functionality specifically for writing papers and presentations
respectively.
Despite papers and presentations having different elements (sections vs
slides) I chose to write the presentations.sty using paper vocabulary to
prevent having to add conditions to automation code.
By adding a `\maketitle` command and using sections instead of slides in
presentation.sty mkdoc can be used without modification for presentation
doctypes.
Similarly, extra macros and logic do not need to be added to any completion
functions written to work with this project for each doctype.
In short, it is not required to keep create the same macros for all
doctypes but it will keep things simple.

**Note:** If you want to make changes to any of the packages above, such as
changing how sections are formatted, I suggest creating a new package that
calls the to-be-modified package and redefining macros (with
`\renewcommand`) and resetting values there or doing the same inside a class.
This will prevent any updates to this project from overwriting your
changes.

Now that we have our desired packages, we can design new classes.
Using the two provided classes as templates should be enough to make new
ones but for more information on creating classes see
[overleaf](https://www.overleaf.com/learn/latex/Writing_your_own_class).

I try to keep classes simple, subclassing a standard class then importing
either the base.sty package or one of its derivatives and setting values
from the base.sty and floatmacros.sty packages.
The values intended to be modified are colors: `\titlecolor`,
`\sectioncolor`, and `\tablebgcolor`; and `\tablewidth`, `\figwidth`.
Colors are set with the `\setcolor{colormacro}{color}` macro and float
widths are set with `\settablewidth` and `setfigwidth`.
These values can be set in packages, classes, or on a project basis.
You may also want to set other package options such as hypertext colors,
section formatting, headers, etc.
Again you have the option of where to set these based on how frequently the
settings are wanted and what type of documents should have them.

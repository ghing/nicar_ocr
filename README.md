# Using OCR to extract data from PDFs

A tutorial on extracting text from PDFs and optical character recognition (OCR) using tesseract, ImageMagick and other open source tools.

This is based on the [tutorial](https://github.com/chadday/nicar_ocr) by Chad Day and updated for the Windows PC labs at NICAR 2020.

## Introduction

This class seeks to help you solve a common problem in journalism: Data stored in a computer generated PDF or even worse an image PDF. We'll first walk through how to do some quick text extraction using a command line tool. Then we'll step up to Optical Character Recognition, or OCR, to work on image files.

While desktop applications such as Adobe Acrobat Pro can do OCR and tools like Tabula can extract tabular data, it's good to be comfortable with command-line tools if you want to a different tool if your go-to produces poor results on a particular PDF, if you need to scale up your PDF processing to a large number of documents or if you want to integrate these tools as part of a more complex processing pipeline.

## Some conventions

We'll use emoji to indicate different kinds of information.

- 🏃: You should run the command in this code cell as part of the tutorial.
- 🍎: Run this version of the command if you're on a Mac.
- 🖥: Run this version of the command if you're on a Windows machine.
- 📕: Don't run the command in this cell. It's just an example for reference.

## Assumptions

- Some comfort with working with the command-line. If you want to better learn the command-line see some of the links in the [resources](#resources) section.
- The tools are installed on your system. See the [installation instructions](#installation) for how to do this if you're trying to walk through this tutorial outside of the NICAR labs.

## Thinking about the command-line

Command-line tools are really different from desktop applicatons. While desktop applications bundle together lots of different functionality into one program, command-line applications try to do one thing and only one thing well. We build functionality by combining command-line tools, using the output of one into the output of another.

All of the options and arguments available to command-line tools can be intimidating, and can make for some pretty long command strings. It's helpful to think about them as the equivalent of the pull-down menus, sliders and checkboxes in a dialog window in a desktop applications. In many cases, you'll just copy and paste the same options every time you run the program. In other cases, you'll need to change the arguments, for things like input and output file names.

Command-line tools don't give you a lot of feedback when the they work as expected. In most cases, no news is good news.

With command-line tools, spaces and capitalization matter. If you run into errors, check that you've retypted the command correctly, or try copying and pasting.

## The tools 

We're using a number of open source software tools to process our PDFS:

* [Xpdf](https://www.xpdfreader.com/) is an open source toolkit to work with pdfs. We'll be using its tool, [pdftotext](https://www.xpdfreader.com/pdftotext-man.html).

* [tesseract](https://github.com/tesseract-ocr/tesseract/wiki) is our OCR engine. It was first developed by HP but for the last decade or so it's been maintained by Google.

* [ImageMagick](http://www.imagemagick.org/script/command-line-processing.php) is an open source image processing and conversion power tool. You can think of it like a command-line photoshop. In this tutorial, we'll be using the `convert` command. As I mentioned earlier, command-line tools do one thing and one thing only, so the command is sort of like the "save as" functionality of Photoshop.

* [Ghostscript](https://www.ghostscript.com/index.html) is an interpreter for PDFs and Adobe's PostScript language. We don't use it directly, but it can be needed by some of the other tools.

## Files

We'll be using a number of files for our examples. You can find them in folders corresponding to each of the three scenarios we'll be walking through together:

- `scenario_one`
- `scenario_two`
- `scenario_three`

## Scenario 1: Analyzing a computer generated pdf with embedded text (searchable pdf)

This is probably the easiest problem to solve dealing with pdfs. We want to extract the text from a searchable pdf for analysis of some type.

There are many GUI software programs you can use to do this. They all have strengths and weaknesses.

* [Cometdocs](https://www.cometdocs.com/)
* [Tabula](https://tabula.technology/) (free and great for tabular data!)
* [Adobe Acrobat Pro](https://acrobat.adobe.com/us/en/acrobat/pricing.html?mv=search&sdid=J7XBWTSV&ef_id=CjwKCAiA1ZDiBRAXEiwAIWyNC62H_xFn3sW5k3JAETpc_MeS9HOq-7l-qD2cvFXcU-Qkl-v_TPYjSxoC4bsQAvD_BwE:G:s&s_kwcid=AL!3085!3!99546333262!e!!g!!%2Badobe%20%2Bacrobat%20%2Bpro&gclid=CjwKCAiA1ZDiBRAXEiwAIWyNC62H_xFn3sW5k3JAETpc_MeS9HOq-7l-qD2cvFXcU-Qkl-v_TPYjSxoC4bsQAvD_BwE) ($$)
* [Abbyy Finereader](https://www.abbyy.com/en-us/finereader/?redirect-from=old-fr-pro&__c=1) ($$ but also very accurate)

For this tutorial, we're going to use an open source powertool from Xpdf called pdftotext. The construction of the command is pretty intuitive. You point it at a file and it outputs a text file.

I often use this tool to check for hidden text, particularly in documents that are redacted. Our example is from just a few months ago when lawyers for Paul Manafort accidentally filed a document that wasn't properly redacted. Reporters, including my colleague Michael Balsamo, quickly realized that even though the document contained blacked out sections, the text of those passages was still present. That text [revealed](https://www.apnews.com/608b9fcbca5941348e2ac8796e94c8cd) Manafort had shared polling data with a Russian associate during the 2016 election.

One way to get to this text is just to copy and paste the sections out. But this can be tedious, particularly if there are a lot of sections or you have a large document. A faster and easier to read method is what we're going to do with Xpdf's pdftotext.

Our [document](files/manafort/Manafort_filing.pdf) has several sections like this.

![Alt Text](/imgs/Manafort_2.png)

But since we can tell that there's text underneath there, let's run it through pdftotext and see what comes out.

🏃 To get started, let's change to the directory for this scenario:

```
cd scenario_one
```

#### pdftotext command construction

📕 `pdftotext` needs to know the path of the PDF file and the path of the output text:

```
pdftotext /path/to/my/file.pdf name-of-my-text-file.txt
```

### Extract text from a PDF

🏃 It's pretty simple to extract all the text in a text PDF to a text file:
 
```
pdftotext Manafort_filing.pdf manafort_filing.txt
```

But that's just one limited use case. Extracting this text can then be fed into databases or used for visualations.

Let's take a look at another one of our files involving tabular data, found [here](/files/tabular/07012018-report-final.pdf). This is a salary roster of Trump White House employees. We'll be using a single image page of this file for a later example.

![Alt Text](/imgs/wh_salaries.png)

As mentioned before, Tabula is a great tool for getting tabular data out of pdf files, but I wanted to give you another option using pdftotext that works well with fixed-width data files like this White House salaries listing. It also has the added benefit of being easily scriptable.

### pdftotext command for tables

📕 The `-table` option tells `pdftotext` to try to extract tabular text from a PDF and maintain the rows/columns:

```
pdftotext -table /path/to/my/file name-of-my-text-file.txt
```

We'll test it out on the [file](/files/tabular/07012018-report-final.pdf).

🏃 Run `pdftotext` with the `-table` option to extract the table.

```
pdftotext -table 07012018-report-final.pdf tabular-test.txt
```

You should get something like this: 

![Alt Text](/imgs/structured.png)

🏃 For comparison, try using just pdftotext without the `-table` option.

```
pdftotext 07012018-report-final.pdf test.txt
```

You should get something like this (very bad stuff):

![Alt Text](/imgs/unstructured.png)

Now that we've walked through the basics of text extraction with computer generated (nice) pdfs, let's go onto the harder use cases.

🏃 Let's change out of the scenario directory so we're ready to move on to the next scenario.

```
cd ..
```

## Scenario 2: Basic text extraction from image files

Extracting text from image files is perhaps one of the most common problems reporters face when they get data from government agencies or are trying to build their own databases from scratch (paper records, the dreaded image pdf of an Excel spreadsheet, etc.) To do this, we use OCR and in this example, Tesseract.

Before we get started, change directoy into the directory for this scenario:

```
cd scenario_two
```

#### Basics of tesseract

🏃 Tesseract has many options. You can see them by typing:

```
tesseract -h
```

We're not going to go into detail on many of these options but you can read more [here](https://github.com/tesseract-ocr/tesseract/wiki)

📕 The basic command structure looks like this:

```
tesseract imagename outputbase [-l lang] [--oem ocrenginemode] [--psm pagesegmode] [configfiles...]
```

Let's look at a single image file. In this case, that's the `wh_salaries.png` file in our imgs folder. This is the first page of our White House salaries pdf but notice that it is not searchable.

This is perhaps the most simple use of tesseract. We will feed in our image file and have it output a searchable pdf.

🏃 Run the `tesseract` command to create a searchable PDF from an image:

```
tesseract wh_salaries.png out pdf
```

You start with a file like this:

![Alt Text](/imgs/wh_salaries.png)

You should get a file name `out.pdf` and you can see that it's searchable.

![Alt Text](/imgs/searchable_salaries.png)

🏃 As with the previous scenario, change out of the scenario directory so we're ready to move on to the next scenario:

```
cd ..
```

## Scenario 3: Combining our skills to make a searchable pdf out of an image pdf.

### Converting pdfs to images to prepare for OCR using ImageMagick

So far, we've covered extracting text from computer generated files and doing some basic OCR. Now, we'll turn to creating searchable pdfs out of image files. To do this, we'll be adding another command line tool called ImageMagick, an image editing and manipulation software.

🏃 Before we get started, change directory to the one for this scenario:

```
cd scenario_three
```

We will be using the `convert` tool from ImageMagick.

ImageMagick has some great documentation that explains all of its many options. You can find it [here](http://www.imagemagick.org/script/command-line-options.php#page).

📕🖥  The general syntax of the `convert` command on Windows is:

```
magick convert [options ...] file [ [options ...] file ...] [options ...] file
```

📕🍎 On a Mac or a Linux system, you can omit the `magick` supercommand: 

```
convert [options ...] file [ [options ...] file ...] [options ...] file
```

If you're familiar with photography or document scanning, you know that the proper image resolution is essential for electronic imaging. When it comes to OCR, this is even more true. 

The general standard for OCR is 300 dpi, or 300 dots per inch, though [ABBYY recommends](https://knowledgebase.abbyy.com/article/489) using 400-600 for font sizes smaller than 10 point. In ImageMagick, this is specified using the density flag. Below we are telling ImageMagick to take our pdf document and convert it to an image with 300 dpi.

### Example with the image file Russia findings document

![Alt Text](/imgs/Screen%20Shot%202019-03-07%20at%208.51.54%20AM.png)

First, we have to convert it to an image so we can run it through tesseract.

🏃🖥  We'll use ImageMagick's `convert` tool.

```
magick convert image_pdf_2.pdf russia_findings.tiff
```

🏃🍎 On a Mac: 

```
convert image_pdf_2.pdf russia_findings.tiff
```

On a Mac, an easy way to find the dpi of an image is to use Preview. Open the image in preview, go to ```Tools``` and click ```Show Inspector```.

So let's take a look at our image we just created.

### Finding the DPI of an image

#### On Windows

This can be obtained through the file properties Window.

Right-click on the file name in Explorer to get the context menu.

Click `Properties` from the context menu.

Click on the `Details` tab.

Scroll down to the `Image` section.

Look for the `Horizontal resolution` and `Vertical resolution` labels.

#### On a Mac

Open in Preview:

![Alt Text](/imgs/preview.png)

Go to 'Show Inspector':

![Alt Text](/imgs/show_inspector.png)

Look in Inspector pane 1:

![Alt Text](/imgs/inspector_1.png)

Look in Inspector pane 2:

![Alt Text](/imgs/inspector_2.png)

### Increasing the DPI

So our dpi is ```72```, which likely is fine for this document but let's go ahead and up that using convert. This will increase the file size of the tiff we create (so warning about file bloat) but it's only a temporary file that we're using to get the best text recognition.

🏃🖥  Let's do this with our Russia document.

```
magick convert -density 300 image_pdf_2.pdf -depth 8 -strip -background white -alpha off russia_findings.tiff
```

🏃🍎 On a Mac, this would be:

```
convert -density 300 image_pdf_2.pdf -depth 8 -strip -background white -alpha off russia_findings.tiff
```

So let's break this down.

`convert` - invokes ImageMagick's convert tool

`-density` - ups the dpi of our image to 300

`russia_finding.pdf` - our file that we're converting to an image.

`-depth 8` - "This the number of bits in a color sample within a pixel. Use this option to specify the depth of raw images whose depth is unknown such as GRAY, RGB, or CMYK, or to change the depth of any image after it has been read", according to ImageMagick documentation.

`-strip` - strips off any junk on the file (profiles, comments, etc.)

`-background white` - sets the background to white to help with contrasting our text

`-alpha off` -generally the transparency of the image. A great explanation [here](https://www.quora.com/What-exactly-is-an-alpha-channel-in-an-image)

#### Now we run this TIFF through tesseract

🏃 Run the extracted image through tesseract to create a searchable text PDF.

```
tesseract russia_findings.tiff -l eng russia_findings_enh pdf
```

And you've got a searchable PDF!

🏃 Let's take a look at the underlying text now.

```
pdftotext russia_findings_enh.pdf russia_text.txt
```

📕 We also could have just outputted directly to a text file like this.

```
tesseract russia_findings.tiff -l eng russia_findings_enh txt
```

## Where to go from here

OCRing is not a perfect science and most of the time, it isn't simple. One recent example: public financial disclosures of federal judges are multi-page documents but they are released as extremely long, single tiff files. You can find a similar test file [here](https://drive.google.com/open?id=11YpC2-0yYyuJL7AJnvG48q9H8hrrDQon)

![Alt Text](/imgs/Walker16.png)

And you'll notice that the pages need to be split.

![Alt Text](/imgs/Walker16_pages.png)

The workflow below walks through one example of how to solve the problem using ImageMagick and Tesseract.

📕 This blows up the images, adjusts the image resolution, ups the contrast to help bring out the text. It then outputs a grayscale version, set at 8-bit depth, named Walker16_enh.tiff.

```
convert -resize 400% -density 450 -brightness-contrast 5x0 Walker16.tiff -set colorspace Gray -separate -average -depth 8 -strip Walker16_enh.tiff
```

Next we use ImageMagick's crop to split it up into a multi-page pdf. 

To find the dimensions, first use Preview's Inspector tool. You 'll see the dimensions of the entire image file. (NOTE: This screenshot is from a different file since I added this later.)

![Alt Text](/imgs/find_inspector.png)

The first value is the width and the second value is the length. To get the pixel length of each page, just divide by the number of pages you should have in the final file.

![Alt Text](/imgs/dimensions.png)

📕 

```
convert Walker16_enh.tiff -crop 3172x4200 Walker16_to_ocr.tiff
```

📕 Then we convert that image into a searchable pdf.

```
tesseract Walker16_to_ocr.tiff -l eng Walker16 pdf
```

Exploring the various options and fine-tuning your skills with ImageMagick can help prepare you for the next big step: Batch processing of documents, which you can hear more about [here at NICAR](https://www.ire.org/events-and-training/event/3433/4227/).

## Installation on your own system

If you want to run the tutorial on your machine, you'll need to install Xpdf, tesseract, ImageMagick and possibly Ghostscript on your computer.

### Mac

For Mac, we'll be using the Homebrew package manager.

📕🍎  To install tesseract, you will use the following command.
```
brew install tesseract
```

📕🍎 For Xpdf, you will use this.
```
brew install xpdf
```

📕🍎 We will also install libtiff, a dependency for ImageMagick that we will need.
```
brew install libtiff
```

📕🍎 Then we'll install ghostscipt
```
brew install ghostscript
```

📕🍎 And for ImageMagick you will use this.
```
brew install imagemagick
```

#### Making sure the TIFF delegate is installed. 

📕🍎 Before we go on from here, let's make sure we have the tiff delegate installed. You can check like this:

```
convert -list configure
```

Scroll down to ```DELEGATES``` and make sure it includes ```tiff```

For example:

```
DELEGATES      bzlib mpeg freetype jng jpeg lzma png tiff xml zlib
```

#### IF you don't have tiff in the list, follow these steps:

📕🍎 First check to make sure that libtiff is installed. You can do this by running

```
brew list
```

📕🍎 If libtiff is not in the list, then install it using brew.
```
brew install libtiff
```

📕🍎 Now check to make sure that imagemagick is recognizing libtiff is installed as a dependency.
```
brew info imagemagick
```

If you're good to go, it should look something like this:

```
==> Dependencies
Build: pkg-config ✔
Required: freetype ✔, jpeg ✔, libheif ✔, libomp ✔, libpng ✔, libtiff ✔, libtool ✔, little-cms2 ✔, openexr ✔, openjpeg ✔, webp ✔, xz ✔
```
Now that we've installed ghostscript and the tiff delegate, let's continue on with our example.

### Windows 

#### Gather the software packages

- Tesseract: http://digi.bib.uni-mannheim.de/tesseract/tesseract-ocr-setup-4.00.00dev.exe
- Ghostscript: https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs922/gs922w64.exe
- Imagemagick: https://imagemagick.org/download/binaries/ImageMagick-7.0.9-26-Q16-x64-dll.exe
- xpdf: https://xpdfreader-dl.s3.amazonaws.com/XpdfReader-win64-4.00.01.exe https://xpdfreader-dl.s3.amazonaws.com/xpdf-tools-win-4.02.zip

These packages are current as of February 2020.  You'll probably want to grab the most recent versions if you're installing this at a later time. You can find more information in the documentation here:

* Xpdf [documentation](https://www.xpdfreader.com/download.html)
* tesseract [documention](https://github.com/tesseract-ocr/tesseract/wiki).
* ImageMagick [documentation](http://www.imagemagick.org/script/command-line-processing.php)
* Ghostscript [documentation](https://ghostscript.com/doc/9.21/Install.htm)

#### Run the installers for the packages

These packages, with the exception of Xpdf, have installers that you will run similar to installing other Windows software.

#### Unzip Xpdf files and move to a reasonable place

The Xpdf command-line tools didn't have an installer. Per the `INSTALL` file in the zip archive, I copied the files to `C:\Program Files\Xpdf`.

#### Add command-line tools to path

You'll want to add the command-line tools to the system's path so I can type `tesseract` instead of having to type out `C:\Program Files (x86)\Tesseract-OCR\tesseract` each time I want to run the program.

The ImageMagick installer adds its command to the path for you.

For Xpdf and tesseract, you'll have to do this manually. Follow [these instructions](https://www.computerhope.com/issues/ch000549.htm#windows10) to add the locations of the software to your system path. On my system, Tesseract was installed to `C:\Program Files (x86)\Tesseract-OCR`.

### Linux

You'll want to install these tools with your particular distribution's package manager. The package names may be slightly different than with Homebrew or Windows.

## Resources

### Learning the command line

Important!: While command line will be very similar on UNIX-based systems like Mac OS X or Linux, they'll vary widely if you're using Windows. Some things are the same (`cd` to change directories), others are different (`ls` vs. `dir` to show directory contents). Just keep these differences in mind when you're searching for additional help.

At NICAR 2020, AJ Vicens gave the workshop [Command line for reporters](https://ireapps.github.io/nicar-2020-schedule#20200307_command_line_for_reporters_mac_2026_all). It covers the use of the command line on Macs.

Good command-line resources for Windows are harder to find. Mike Stucka made [this tipsheet](https://www.ire.org/resource-center/tipsheets/4956/) for a 2017 NICAR session on the Windows command line.
[This one](https://www.cs.princeton.edu/courses/archive/spr05/cos126/cmd-prompt.html) is also pretty good. 

## Sources and references

I created this tutorial for [NICAR 2019]('https://www.ire.org/events-and-training/conferences/nicar-2019') but it relies on many helpful open source resources that deserve credit. They are listed below. Thanks for sharing your work with the rest of the world.

- [Tesseract](https://github.com/tesseract-ocr/tesseract/wiki/Command-Line-Usage) documentation
- [ImageMagick](https://www.imagemagick.org/script/command-line-processing.php) documentation
- [pdftotext](https://www.xpdfreader.com/pdftotext-man.html) documentation

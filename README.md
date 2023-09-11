# Digital Scholarly Editions Static Site Cookiecutter: Project

This project had the aim to set up a static website using the dse-static-cookiecutter framework, in order to publish XML documents as static html pages. This particular project has focused on one XML document in particular (msDesc_417389.xml) and used XSLT stylesheets to transform it to HTML.

### Challenges faced
#### Linux installation
- Ubuntu 23.04 originally installed <-- unfortunately incompatible with pip, and pipx would not produce the same results
- WSL (Windows Services for Linux) <-- Python pip installation could not proceed
- Fedora (latest version) <-- Python would not work
- Rocky Linux 8.8 (RHEL) <-- issues with privileges
- Ubuntu 22.04 LTS <-- ultimately worked, but shellscripts did not work until late Friday evening (08.09.23)

NB: All images (except WSL) were run in Oracle Virtual Box.

#### Issue with scripts
The shellscript invoked to install Saxon was not working due to a delay in delivering files from the sourceforge site. The ```wget``` command was not appropriate for that scenario, and I tried using ```curl``` without success. I also downloaded Saxon from Saxonica and installed it, but it still did not work as intended.

#### Issue with GitHub: password authentication
Password authentication is no longer supported when synchronising from local git source to GitHub. The solution was to use an SSH-based connection.

## Installation and Steps
1) Install default Ubuntu 22.04 LTS
- Memory size: 8GB
- vCPU: 2
- Disk: 100GB (can be smaller)

2) Update and upgrade operating system
```
sudo apt update

sudo apt upgrade
```

3) Install Python and pip
```
sudo apt install python3-full

sudo apt install python3-pip
```

4) Install cookiecutter
```
pip install -U cookiecutter
```

5) Install ant, ant-contrib, and ant-contrib-cpptasks
```
sudo apt install ant

sudo apt install ant-contrib

sudo apt install ant-contrib-cpptasks
```

6) Install git
```
sudo apt install git
```

7) Create git user account and create repository

8) Generate a new dse-static-site project
```
cookiecutter gh:acdh-oeaw/dse-static-cookiecutter
```
- Select all default options except for question 10/10 --> choose option 2: staticsearch

9) Move to dse-static directory and run script
```
cd dse-static

./shellscripts/script.sh
```

10) Build site
```
ant
```

11) Build index
```
./shellscripts/build_index.sh
```

12) Sign into GitHub and create and deploy your SSH key
- Create SSH key
  * Go to home directory and run: ```ssh-keygen -o```
- Deploy SSH key
  * Go to .ssh directory: ```cd .ssh```
  * Copy public key ```cat id_rsa.pub``` to GitHub (Settings/Security/Deploy keys)

13) Create local git
- From home directory, issue command: ```cd dse-static```
- Then create an empty git repository: ```git init```

14) Modify .gitignore to avoid saving changes to HTML files
- Use editor to comment the line: *.html (# *.html)

15) Configure local git
```
git config --global user.email "katnew197@gmx.at"

git config --global user.name "KatBishop23"
```

16) Modify file and commit to local git
- Alter README.md file: ```echo "# ACDH-CH" >> README.md```
- Commit to local git
```
git add README.md

git commit -m "first successful commit"
```

17) Create local git branch
```
git branch -M main
```

18) Configure GitHub location as origin
```
git remote add origin git@github.com:KatBishop23/ACDH-CH.git
```

19) Push local branch to GitHub
```
git push -u origin main
```

20) Download [msDesc_417389.xml](https://arche.acdh.oeaw.ac.at/api/428369) and copy to ```data/editions```

21) Run ant: ```ant```
- You can see the new document listed in the table of contents in html/toc.html
- You can see the webpage for it at html/msDesc_417389.html

22) Once you are ready, add all files to local git and commit changes
```
git add --all

git commit -m "second successful commit"
```

23) Push to GitHub
```
git push -u origin main
```

24) Adapt XSLT Stylesheet to new input data
[SEE _ADAPT XSLT TO INPUT DATA_ BELOW]

25) Modify .gitignore
- Use the editor to remove the comment in front of *.html

26) Add HTML files into local git repository and commit changes
```
git add --all

git commit -m "with html commit"
```

27) Push to GitHub
```
git push -u origin main
```

## Adapt XSLT to new input data
### Adapt the stylesheet xslt/editions.xsl to render the full ```<tei:teiHeader>```
Above ```<div class="card-body">```, add the following to ensure the header is rendered in the HTML file:
```
<div class="card-header">
    <xsl:apply-templates select=".//tei:teiHeader"></xsl:apply-templates>
</div>
```

### Adapt the stylesheet to render the output of the ```<tei:msDesc>``` element and its descendants (e.g. msIdentifier)
I tried rendering the element in a few ways, but did not manage to render it in my several attempts. I believe the issue may be related to my use of XPath to select the correct nodes.

My approach was to create a table for the elements in msIdentifier.

The final version:
```
<div class="card-header">
    <xsl:apply-templates select=".//tei:teiHeader"></xsl:apply-templates>
        <html>
            <ms_Desc>
                <h2>MS Identifier</h2>
                <table border="1">
    <tr bgcolor="#f0be0a">
    <th style="text-align:centre">Country</th>
    <th style="text-align:centre">Settlement</th>
    <th style="text-align:centre">Institution</th>
    <th style="text-align:centre">Repository</th>
    <th style="text-align:centre">ID Number</th>
    </tr>
    <xsl:for-each select="/TEI/teiHeader[1]/fileDesc[1]/sourceDesc[1]/msDesc[1]/msIdentifier[1]">
        <tr>
        <td><xsl:value-of select="/TEI/teiHeader[1]/fileDesc[1]/sourceDesc[1]/msDesc[1]/msIdentifier[1]/country[1]" /></td>
        </tr>
        <tr>
        <td><xsl:value-of select="/TEI/teiHeader[1]/fileDesc[1]/sourceDesc[1]/msDesc[1]/msIdentifier[1]/settlement[1]" /></td>
        </tr>
        <tr>
        <td><xsl:value-of select="/TEI/teiHeader[1]/fileDesc[1]/sourceDesc[1]/msDesc[1]/msIdentifier[1]/institution[1]" /></td>
        </tr>
        <tr>
        <td><xsl:value-of select="/TEI/teiHeader[1]/fileDesc[1]/sourceDesc[1]/msDesc[1]/msIdentifier[1]/repository[1]" /></td>
        </tr>
        <tr>
        <td><xsl:value-of select="/TEI/teiHeader[1]/fileDesc[1]/sourceDesc[1]/msDesc[1]/msIdentifier[1]/idno[1]" /></td>
        </tr>
        </xsl:for-each>
        </table>
        </ms_Desc>
        </html>               
</div>
```
This created a table (header row), but did not insert the values from the XML file. Since the code had worked with other XML files with which I experimented, I believe it was an issue with the identification of the correct node using XPath.

I also noted that the text that was part of the ```<msDesc>``` element had already been included as part of the header in general, so I tried to exclude ```<msDesc>``` in the header (editions4.xsl) so that I could include the text separately using my table, but this unfortunately also did not work:
```
<div class="card-header">
    <xsl:apply-templates select=".//tei:teiHeader[not(./tei:msDesc)]"></xsl:apply-templates>
    ...
</div>
```

### Once you are done...
Do not forget to re-run the ant transformation and commit the new output.

## Credits
- Followed [acdh-oeaw’s](https://github.com/acdh-oeaw) instructions and built with [DSE-Static-Cookiecutter](https://github.com/acdh-oeaw/dse-static-cookiecutter)
- msDesc_417389.xml file obtained [here](https://arche.acdh.oeaw.ac.at/api/428369)

# ACDH-CH 

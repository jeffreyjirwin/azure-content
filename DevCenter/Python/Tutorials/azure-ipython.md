# IPython Notebook on Windows Azure


For a quick overview of installation and IPython, please watch:

<div class="dev-video-thumb" style="background-image:url(../Media/ipy-youtube2.png)"><a href="http://go.microsoft.com/fwlink/?LinkId=254535">IPython Overview</a></div>


The [IPython project](http://ipython.org) provides a collection of tools for
scientific computing that include powerful interactive shells, high-performance
and easy to use parallel libraries and a web-based environment called the
IPython Notebook. The Notebook provides a working environment for interactive
computing that combines code execution with the creation of a live
computational document. These notebook files can contain arbitrary text,
mathematical formulas, input code, results, graphics, videos and any other kind
of media that a modern web browser is capable of displaying.

Whether you're absolutely new to Python and want to learn it in a fun,
interactive environment or do some serious parallel/technical computing, the
IPython Notebook is a great choice. As an illustration of its capabilities, the
following screenshot shows the IPython Notebook being used, in combination with
the SciPy and matplotlib packages, to analyze the structure of a sound
recording:

![Screenshot](../Media/ipy-notebook-spectral.png)

This document will show you how to deploy the IPython Notebook on Microsoft
Azure, using Linux or Windows virtual machines (VMs).  By using the IPython
Notebook on Azure, you can easily provide a web-accessible interface to
scalable computational resources with all the power of Python and its many
libraries.  Since all installation is done in the cloud, users can access these
resources without the need for any local configuration beyond a modern web
browser.

<div chunk="../../Shared/Chunks/create-account-and-vms-note.md" />

## Create and Configure a VM on Azure

The first step is to create a Virtual Machine (VM) instance running on Azure.
This VM is a complete operating system in the cloud and will be used to
run the IPython Notebook. Windows Azure is capable of running both Linux and Windows
virtual machines, and we will cover the setup of IPython on both types of virtual machines.

### Linux VM

To create a Linux VM, click on the "New" icon at the bottom left of the 
Azure Management Portal and then select "Virtual Machine" from the list
of options. For our Linux VM we are going to use OpenSUSE, the selection
of which can be seen in the following screenshot of the "VM OS Selection"
dialog.

![Screenshot](../Media/ipy-azure-linux-001.png)

On the next configuration screen we name our VM (`ipython-demo`), pick an
administrator username (`ipadmin`) and password. With this user, we'll later be
able to SSH into the VM to install IPython. This is shown in the "VM
Configuration" dialog:

![Screenshot](../Media/ipy-azure-linux-002.png)

Next, on the "VM Mode" dialog use the default options, but give the VM a
public DNS name, which we've chosen to be `ipython-demo`:

![Screenshot](../Media/ipy-azure-linux-003.png)

With this choice, the instance will be available online as
`ipython-demo.cloudapp.net`. Once the VM has started the dashboard will appear
as follows:

![Screenshot](../Media/ipy-azure-linux-004.png)

### Windows VM

The creation of the Windows VM is nearly identical to the creation of the Linux 
VM described above. The first main difference is in the "VM OS Selection" 
dialog, where we select "Windows Server 2008 R2 SP1, March 2012" as shown in 
the following screenshot:

![Screenshot](../Media/ipy-azure-win-001.jpg)

After that, follow the same steps described above for Linux.

The only other difference is in how you will connect to the VM. Instead of 
using SSH, you will use Microsoft's Remote Desktop application. Thankfully, the 
Azure Management Portal provides an `.rpd` file that contains the connection 
information to connect to the VM. To download this file, click on the "Connect" 
link at the bottom of the VM Dashboard. Once you have downloaded the `.rdp` 
file simply open it in Remote Desktop to connect to the VM.

## Create an Endpoint for the IPython Notebook

This step applies to both the Linux and Windows VM. Later on we will configure
IPython to run its notebook server on port 9999. To make this port publicly
available, we must create an endpoint in the Windows Azure Management Portal. This
endpoint opens up a port in the Windows Azure firewall and maps the public port (HTTPS,
443) to the private port on the VM (9999).

To create an endpoint, go to the VM dashboard, click "Endpoints", then "Add
Endpoint" and create a new endpoint (called `ipython_nb` in this example). Pick
TCP for the protocol, 443 for the public port and 9999 for the private port:

![Screenshot](../Media/ipy-azure-linux-005.png)

After this step, the "Endpoints" Dashboard tab will look like this:

![Screenshot](../Media/ipy-azure-linux-006.png)

## Install Required Software on the VM

To run the IPython Notebook on our VM, we must first install IPython and
its dependencies.

### Linux

To install IPython and its dependencies, SSH into the Linux VM and carry out 
the following steps.

1.  Install [NumPy][numpy], [Matplotlib][matplotlib], [Tornado][tornado] and
    other IPython's dependencies by doing:

        sudo zypper install python-matplotlib
        sudo zypper install python-tornado
        sudo zypper install ipython

2.  Download and install the latest version of IPython by doing:

        sudo easy_install http://github.com/ipython/ipython/tarball/master

### Windows

To install IPython and its dependencies on the Windows VM, use the `.rdp` file 
and Remote Desktop to connect to the VM. Then carry out the following steps, 
using the Windows PowerShell to run all command line actions.

1.  Install Python 2.7.2 (32 bit) from http://python.org. You will also need to
    add `C:\Python27` and `C:\Python27\Scripts` to your `PATH` environment
    variable.

2.  Install distribute by downloading the file 
    http://python-distribute.org/distribute_setup.py and then running the
    command:

        python distribute_setup.py

3.  Install [Tornado][tornado] and [PyZMQ][pyzmq] by running the commands:

        easy_install tornado
        easy_install pyzmq

4.  Download and install [NumPy][numpy] and [Matplotlib][matplotlib] using the
    `.exe` binary installers available on their respective websites.

5.  Download and install OpenSSL. You will need to install both the 
    "Win32 OpenSSL v1.0.1c Light" and "Visual C++ 2008  Redistributable" from
    http://slproweb.com/products/Win32OpenSSL.html. You will also need to add
    `C:\OpenSSL-Win32\bin` to your `PATH` environment variable.

5.  Download and install the latest version of IPython by doing:

        easy_install http://github.com/ipython/ipython/tarball/master

### Configure the IPython Notebook

Next, we configure the IPython Notebook. The first step is to create a custom
IPython configuration profile to encapsulate the configuration information:

    ipython profile create nbserver

Next we `cd` to the profile directory to create our SSL certificate and edit
the profiles configuration file.

On Linux:

    cd ~/.config/ipython/profile_nbserver/

On Windows:

    cd ~\.ipython\profile_nbserver

On both platforms create the SSL certificate as follows:

    openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem

Note that since we are creating a self-signed SSL certificate, when connecting
to the notebook your browser will give you a security warning.  For long-term
production use, you will want to use a properly signed certificate associated
with your organization.  Since certificate management is beyond the scope of
this demo, we will stick to a self-signed certificate for now.

In addition to using a certificate, you must also provide a password to protect
your notebook from unauthorized use.  For security reasons IPython uses
encrypted passwords in its configuration file, so you'll need to encrypt your
password first.  IPython provides a utility to do so; at a command prompt run:

    python -c "import IPython;print IPython.lib.passwd()"

This will prompt you for a password and confirmation, and will then print the
password as follows:

    Enter password: 
    Verify password: 
    sha1:b86e933199ad:a02e9592e59723da722.. (elided the rest for security)
    
Next, we will edit the profile's configuration file, which is the
`ipython_notebook_config.py` file in the profile directory you are in.  This
file has a number of fields and by default all are commented out.  You can open
this file with any text editor of your liking, and you should ensure that it
has at least the following content:

     c = get_config()

     # This starts plotting support always with matplotlib
     c.IPKernelApp.pylab = 'inline'

     # You must give the path to the certificate file.

     # If using a Linux VM:
     c.NotebookApp.certfile = u'/home/ipadmin/.config/ipython/profile_nbserver/mycert.pem'

     # And if using a Windows VM:
     c.NotebookApp.certfile = r'C:\Users\Administrator\.ipython\profile_nbserver\mycert.pem'

     # Create your own password as indicated above
     c.NotebookApp.password = u'sha1:b86e933199ad:a02e9592e5 etc... '

     # Network and browser details. We use a fixed port (9999) so it matches
     # our Windows Azure setup, where we've allowed traffic on that port
     
     c.NotebookApp.ip = '*'
     c.NotebookApp.port = 9999
     c.NotebookApp.open_browser = False


### Run the IPython Notebook

At this point we are ready to start the IPython Notebook. To do this,
navigate to the directory you want to store notebooks in and start
the IPython Notebook server:

    ipython notebook --profile=nbserver

You should now be able to access your IPython Notebook at the address
`https://[Your Chosen Name Here].cloudapp.net`.

When you first access your notebook, the login page asks for your password:

![Screenshot](../Media/ipy-notebook-001.png)

And once you log in, you will see the "IPython Notebook Dashboard", which is
the control center for all notebook operations.  From this page you can create
new notebooks, open existing ones, etc:

![Screenshot](../Media/ipy-notebook-002.png)

If you click on the "New Notebook" button, you will see an opening page as
follows:

![Screenshot](../Media/ipy-notebook-003.png)

The area marked with an `In []:` prompt is the input area, and here you can
type any valid Python code and it will execute when you hit `Shift-Enter` or
click on the "Play" icon (the right-pointing triangle in the toolbar).

Sinc we have configured the notebook to start with NumPy and matplotlib support
automatically, you can even produce figures, for example:

![Screenshot](../Media/ipy-notebook-004.png)

## A powerful paradigm: live computational documents with rich media

The notebook itself should feel very natural to anyone who has used Python and
a word processor, because it is in some ways a mix of both: you can execute
blocks of Python code, but you can also keep notes and other text by changing
the style of a cell from "Code" to "Markdown" using the drop-down menu in the
toolbar:

![Screenshot](../Media/ipy-notebook-005.png)


But this is much more than a word processor, as the IPython notebook allows the
mixing of computation and rich media (text, graphics, video and virtually
anything a modern web browser can display). For example, you can mix
explanatory videos with computation for educational purposes:

![Screenshot](../Media/ipy-notebook-006.png)

or embed external websites that remain live and usable, inside of a notebook
file:

![Screenshot](../Media/ipy-notebook-007.png)

And with the power of Python's many excellent libraries for scientific and
technical computing, a simple calculation can be performed with the same ease
than a complex network analysis, all in one environment:

![Screenshot](../Media/ipy-notebook-008.png)

This paradigm of mixing the power of the modern web with live computation
offers many possibilities, and is ideally suited for the cloud; the Notebook
can be used:

* as a computational scratchpad to record exploratory work on a problem,

* to share results with colleagues, either in 'live' computational form or in
  hardcopy formats (HTML, PDF),

* to distribute and present live teaching materials that involve computation,
  so students can immediately experiment with the real code, modify it and
  re-execute it interactively,

* to provide "executable papers" that present the results of research in a way
  that can be immediately reproduced, validated and extended by others,

* as a platform for collaborative computing: multiple users can log into the
  same notebook server to share a live computational session,

* and more...

If you go to the IPython source code repository, you will find an entire
directory with [notebook
examples](https://github.com/ipython/ipython/tree/master/docs/examples/notebooks)
which you can download and then experiment with on your own Azure IPython VM.
Simply download the `.ipynb` files from the site and upload them onto the
dashboard of your notebook Azure VM (or download them directly into the VM).

## Conclusion

The IPython Notebook provides a powerful interface for accessing interactively
the power of the Python ecosystem on Windows Azure.  It covers a wide range of
usage cases including simple exploration and learning Python, data analysis and
visualization, simulation and parallel computing. The resulting Notebook
documents contain a complete record of the computations that are performed and
can be shared with other IPython users.  The IPython Notebook can be used as a
local application, but it is ideally suited for cloud deployments on Azure

The core features of IPython are also available inside Visual Studio via the 
[Python Tools for Visual Studio](http://pytools.codeplex.com) (PTVS). PTVS is a free and open-source plug-in 
from Microsoft that turns Visual Studio into an advanced Python development 
environment that includes an advanced editor with IntelliSense, debugging, 
profiling and parallel computing integration.


[ipython]:      http://ipython.org                  "IPython"
[tornado]:      http://www.tornadoweb.org/          "Tornado"
[PyZMQ]:        https://github.com/zeromq/pyzmq     "PyZMQ"
[NumPy]:        http://numpy.scipy.org/             "NumPy"
[Matplotlib]:   http://matplotlib.sourceforge.net/  "Matplotlib"

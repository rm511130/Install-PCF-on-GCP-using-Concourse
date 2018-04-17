# How to deploy PCF 2.0 on GCP using a Bosh-managed Concourse CI/CD Pipeline - April 2018
The instructions on this page were taken from https://github.com/cloudfoundry-incubator/bosh-google-cpi-release/tree/master/docs/concourse and enhanced with examples and diagrams so that anyone, from newbies to experts, may get PCF up and running on their 1st attempt. My goal is to keep you from struggling with cryptic formats and hidden details that normal instructions seem to skip. 
As a result, these instructions will become outdated as newer versions of [PCF](https://pivotal.io/de/platform), [Concourse](http://concourse-ci.org/), [GCE](https://cloud.google.com/) and [Bosh](http://bosh.io) become available.

Notes: 
* my workstation is a  Mac, so my examples may need to be adapted if you plan on using a Windows workstation
* when you see `$ terraform version`, it means you need to execute the command that follows the `$` prompt

# We start by deploying Concourse on GCP using Bosh
We will deploy a BOSH director v1.x as part of these instructions. This Bosh Director will be dedicated to monitor and manage the health of the Concourse installation and it should not be used for a PCF deployment. We will need a separate BOSH v2.x to install PCF.

<details>
  <summary>Click to see a Bosh-Director diagram:</summary>

![](./Bosh-Director.png)

</details>

## List of Pre-requisites
Don't take any action yet. This list was placed here so you can get an idea of the tools we will be using.
* An account on [Google Cloud Platform](https://cloud.google.com/)
* An account on [Github](https://github.com/)
* The [Terraform CLI](https://www.terraform.io/intro/index.html) installed on your workstation
* [Python 2.7.x](https://www.python.org/downloads/) installed on your workstation. The version is important. You need 2.7.x and not 3.x
* The [gcloud CLI](https://cloud.google.com/sdk/gcloud/) installed on your workstation

We will walk through each one of the items listed above to make sure all the necessary dependencies are in place.

### Accessing the GCP (Google Cloud Platform) Console 
To access the GCP Console you need a GCP account at https://cloud.google.com

![](./gcp-login.png)

Once you are logged in, you should see a GCP Console similar to the example shown below:

![](./gcp-console.png)

Well done! You have met the 1st pre-requisite.

### Installation of the Terraform CLI 
We will need to use the [Terraform](https://www.terraform.io/intro/index.html) CLI from your workstation, so let's verify if `Terraform` is already installed:
```
$ terraform version
Terraform v0.11.7
```
If you don't already have `Terraform` you can download and unzip [Terraform](https://www.terraform.io/downloads.html) to a directory of your chosing and then symlink the terraform executable to your `/usr/bin`. If you use `Brew` You can `brew install terraform` or `brew upgrade terraform` depending on whether or not you already have it installed.

<details>
  <summary>Click to expand</summary>

Here's what happened on my  Mac when I executed `brew upgrade terraform`:

```
$ brew upgrade terraform
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 3 taps (caskroom/cask, cloudfoundry/tap, homebrew/core).
==> New Formulae
annie              cquery             hcloud             libdazzle          php-code-sniffer   rtptools           tox
apache-arrow-glib  flintrock          jrtplib            meson-internal     php-cs-fixer       scrcpy             trezor-agent
aws-es-proxy       goto               jsonrpc-glib       mint               phpunit            ssh-permit-a38     util-linux
composer           gr-osmosdr         latexdiff          netdata            qsoas              template-glib
==> Updated Formulae
awscli ✔                          fades                             libdvdcss                         picard-tools
cloudfoundry/tap/bbl ✔            fail2ban                          libepoxy                          pick
cloudfoundry/tap/cf-cli ✔         fd                                libfreehand                       pipenv
cloudfoundry/tap/credhub-cli ✔    fdroidserver                      libgit2                           pipes-sh
go ✔                              ffmbc                             libgit2-glib                      planck
godep ✔                           firebase-cli                      libgweather                       plantuml
hugo ✔                            fits                              libhdhomerun                      pmd
openssl ✔                         flatbuffers                       libhttpseverywhere                points2grid
python ✔                          flawfinder                        libidn                            postgis
redis ✔                           flow                              liblas                            pqiv
sqlite ✔                          fluent-bit                        liblcf                            pre-commit
terraform ✔                       fn                                libmpdclient                      presto
abcmidi                           folly                             libmspub                          primesieve
abnfgen                           fonttools                         libmtp                            proj
agda                              fq                                libosmium                         psqlodbc
agedu                             freeciv                           libphonenumber                    pugixml
allegro                           freeling                          libqalculate                      pycodestyle
amber                             freetds                           librdkafka                        pyenv
ammonite-repl                     frege                             libressl                          pygobject3
angular-cli                       frugal                            librsvg                           qbs
ansible                           fuse-emulator                     libsecret                         qpid-proton
ant                               futhark                           libsoup                           qscintilla2
apache-arrow                      g3log                             libspectre                        quicktype
apache-geode                      gcab                              libtensorflow                     raylib
apibuilder-cli                    gdal                              libusb                            re2
apktool                           gdcm                              libuv                             reminiscence
arangodb                          gdk-pixbuf                        libvirt                           robot-framework
armor                             geckodriver                       libvisio                          rocksdb
artifactory                       gedit                             linkerd                           rom-tools
asciinema                         gegl                              lldpd                             ruby
asio                              get-flash-videos                  lmdb                              ruby-build
at-spi2-atk                       get_iplayer                       lmod                              ruby@2.2
at-spi2-core                      geth                              lolcat                            ruby@2.3
atk                               getmail                           lua                               rust
ats2-postiats                     ghostscript                       lua@5.1                           s-nail
autopep8                          git                               lxc                               s6
aws-sdk-cpp                       git-annex                         mackup                            sagittarius-scheme
azure-cli                         git-ftp                           macvim                            saltstack
babl                              git-town                          mame                              samtools
ballerina                         gitbucket                         mapnik                            sbcl
basex                             gitfs                             mapserver                         sbt
bcftools                          gitg                              mariadb                           scipy
bcpp                              gitless                           mariadb@10.1                      scm-manager
bedops                            gjs                               maxwell                           sdb
bettercap                         glade                             mbedtls                           sdlpop
bit                               glib                              media-info                        sile
bitcoin                           glibmm                            mediaconch                        simgrid
bitrise                           gmt                               mednafen                          simple-tiles
botan                             gmt@4                             memcached                         singular
braid                             gnatsd                            mercurial                         sip
brotli                            gnome-builder                     metabase                          sjk
buku                              gnu-sed                           mill                              skaffold
byteman                           gnupg                             mimic                             skafos
bzt                               go-jira                           minio-mc                          skinny
c14-cli                           go@1.9                            miniupnpc                         slimerjs
cabal-install                     gobject-introspection             mitmproxy                         solr
caddy                             goenv                             mkvtoolnix                        sourcekitten
calicoctl                         gom                               mm-common                         sourcery
ccache                            google-benchmark                  modules                           sox
ceres-solver                      gosu                              monetdb                           spigot
certbot                           grafana                           mongo-c-driver                    sqldiff
chakra                            grails                            mongodb@3.4                       sqlite-analyzer
chamber                           grakn                             mpd                               sqlmap
chapel                            graphene                          multimarkdown                     srt
checkstyle                        gron                              mypy                              ssh-copy-id
chromedriver                      groonga                           nano                              sshuttle
chronograf                        grpc                              nativefier                        stdman
clamav                            gsoap                             nats-streaming-server             stellar-core
cliclick                          gst-plugins-good                  ncmpc                             svgcleaner
clinfo                            gtk-doc                           nco                               swiftlint
closure-compiler                  gtk-vnc                           neo4j                             syncthing
cmake                             gtksourceview3                    neofetch                          sysbench
cnats                             gutenberg                         neomutt                           sysdig
cockroach                         hadolint                          newsboat                          talloc
cocoapods                         handbrake                         nginx                             tarantool
coffeescript                      haproxy                           nifi                              tbb
collectd                          harfbuzz                          node                              tectonic
conan                             headphones                        node-build                        teleport
conjure-up                        heroku                            node@4                            terragrunt
container-diff                    hfstospell                        node@6                            testssl
convmv                            hledger                           node@8                            tinyxml2
convox                            hlint                             nodeenv                           tmuxinator-completion
couchdb                           htmldoc                           noti                              todo-txt
cppcheck                          htop                              notmuch                           tomcat
cryptopp                          htslib                            nss                               treefrog
czmq                              http-parser                       ntl                               twarc
darcs                             hwloc                             nuget                             txr
dartsim                           icu4c                             nuxeo                             typescript
davmail                           imagemagick                       ocrmypdf                          uhd
dbhash                            imagemagick@6                     odpi                              unbound
dcm2niix                          internetarchive                   offlineimap                       unrar
dcos-cli                          ipfs                              open-mpi                          unyaffs
ddgr                              ipython                           open-scene-graph                  vala
dependency-check                  ipython@5                         openimageio                       vault
dhall-json                        ircd-hybrid                       openldap                          vim
di                                jadx                              openshift-cli                     vim@7.4
diffoscope                        jbig2enc                          openssh                           vips
django-completion                 jena                              openssl@1.1                       vis
dnscrypt-proxy                    jenkins                           openttd                           vnu
docfx                             jenkins-job-builder               openvdb                           vte3
docker                            jhipster                          optipng                           watch
docker-completion                 joplin                            osm2pgrouting                     webpack
docker-compose                    jpegoptim                         osmium-tool                       widelands
docker-compose-completion         json-glib                         osquery                           wine
doctl                             juju                              packer                            wireshark
dovecot                           kedge                             packmol                           xmount
dscanner                          knot-resolver                     paket                             xmrig
duck                              kompose                           pandoc-crossref                   xtensor
dwdiff                            kotlin                            pango                             yash
dynamips                          kubectx                           parallelstl                       yaz
e2fsprogs                         kubeless                          passenger                         youtube-dl
ejabberd                          kubernetes-cli                    pazpar2                           yq
embulk                            languagetool                      pcb                               zabbix
erlang@19                         laszip                            pdal                              zebra
etcd                              lbdb                              pdnsrec                           zenity
evince                            libatomic_ops                     percona-server-mongodb            zeromq
exomizer                          libbi                             percona-xtrabackup                zorba
exploitdb                         libcdr                            pgcli                             zsh
eye-d3                            libcouchbase                      pgroonga                          zstd
faas-cli                          libdill                           php
==> Renamed Formulae
php56 -> php@5.6                             php70 -> php@7.0                             php71 -> php@7.1
==> Deleted Formulae
arm

==> Upgrading 1 outdated package, with result:
terraform 0.11.5 -> 0.11.7
==> Upgrading terraform 
==> Installing dependencies for terraform: go
==> Installing terraform dependency: go
==> Downloading https://dl.google.com/go/go1.10.1.src.tar.gz
######################################################################## 100.0%
==> Downloading https://storage.googleapis.com/golang/go1.7.darwin-amd64.tar.gz
Already downloaded: /Users/rmeira/Library/Caches/Homebrew/go--gobootstrap-1.7.tar.gz
==> ./make.bash --no-clean
==> /usr/local/Cellar/go/1.10.1/bin/go install -race std
==> Cloning https://go.googlesource.com/tools.git
Updating /Users/rmeira/Library/Caches/Homebrew/go--gotools--git
==> Checking out branch release-branch.go1.10
==> go build
==> Caveats
A valid GOPATH is required to use the `go get` command.
If $GOPATH is not specified, $HOME/go will be used by default:
  https://golang.org/doc/code.html#GOPATH

You may wish to add the GOROOT-based install location to your PATH:
  export PATH=$PATH:/usr/local/opt/go/libexec/bin
==> Summary
🍺  /usr/local/Cellar/go/1.10.1: 8,158 files, 336.8MB, built in 1 minute 41 seconds
==> Installing terraform
==> Downloading https://github.com/hashicorp/terraform/archive/v0.11.7.tar.gz
==> Downloading from https://codeload.github.com/hashicorp/terraform/tar.gz/v0.11.7
######################################################################## 100.0%
==> Cloning https://go.googlesource.com/tools.git
Updating /Users/rmeira/Library/Caches/Homebrew/terraform--golang.org-x-tools--git
==> Checking out branch release-branch.go1.10
==> go install
==> make test bin
🍺  /usr/local/Cellar/terraform/0.11.7: 6 files, 80.2MB, built in 2 minutes 56 seconds
```
  whatever
</details>

Well done! You have the Terraform CLI installed. Another pre-requisite has been met.

### Installation of Python 2.7

* Pyhton 2.7.x is a pre-requisite for the installation of the gcloud CLI, so you need to check whether you have Python

_using my  Mac_
```
$ python -V
Python 2.7.8
```

Lucky me, I already have a good version of Python 2.7.x

* If you don't have Python 2.7.x, you will need to download it from https://www.python.org/downloads/ and follow the instructions to install it.

### Installation gcloud CLI on your GCP account

* In order to control from scripts what goes on in your GCP Account, you must have the `gcloud` CLI installed on your workstation. 
* Download it from [cloud.google.com/sdk](https://cloud.google.com/sdk/).
* On a Mac, downloaded files end up at `~/Downloads/`, so I'm going to use Mac's Finder App to unzip the `google-cloud-sdk-194.0.0-darwin-x86_64.tar.gz` file by simply double-clicking on it.

![](./google-cloud-sdk-download.png)

* Per the example shown above, I now have a `~/Downloads/google-cloud-sdk/` directory which I'm going to move over to my `/work/` directory:

_using my  Mac_
````
$ cd /work                              # this is where I chose to place my the gcloud CLI software
$ mv ~/Downloads/google-cloud-sdk .
$ cd google-cloud-sdk/
$ ls -las
total 608
  0 drwxr-xr-x@  21 rmeira  staff     714 Apr  6 11:02 .
  0 drwxr-xr-x  206 rmeira  staff    7004 Apr 11 12:20 ..
  0 drwxr-xr-x@  21 rmeira  staff     714 Apr  6 11:02 .install
  8 -rw-r--r--@   1 rmeira  staff     980 Apr  6 10:57 LICENSE
  8 -rw-r--r--@   1 rmeira  staff     673 Apr  6 10:57 README
520 -rw-r--r--@   1 rmeira  staff  262408 Apr  6 10:57 RELEASE_NOTES
  8 -rw-r--r--@   1 rmeira  staff       8 Apr  6 10:58 VERSION
  0 drwxr-xr-x@  11 rmeira  staff     374 Apr  6 10:58 bin
  8 -rw-r--r--@   1 rmeira  staff    2711 Apr  6 10:57 completion.bash.inc
  8 -rw-r--r--@   1 rmeira  staff    2083 Apr  6 10:57 completion.zsh.inc
  0 drwxr-xr-x@   3 rmeira  staff     102 Apr  6 11:02 data
  0 drwxr-xr-x@   3 rmeira  staff     102 Apr  6 11:02 deb
  8 -rwxr-xr-x@   1 rmeira  staff    1996 Apr  6 10:57 install.bat
  8 -rwxr-xr-x@   1 rmeira  staff    3701 Apr  6 10:57 install.sh
  0 drwxr-xr-x@   7 rmeira  staff     238 Apr  6 10:57 lib
  8 -rw-r--r--@   1 rmeira  staff     324 Apr  6 10:57 path.bash.inc
  8 -rw-r--r--@   1 rmeira  staff    1210 Apr  6 10:57 path.fish.inc
  8 -rw-r--r--@   1 rmeira  staff      31 Apr  6 10:57 path.zsh.inc
  0 drwxr-xr-x@   5 rmeira  staff     170 Apr  6 11:02 platform
  8 -rw-r--r--@   1 rmeira  staff      39 Apr  6 11:02 properties
  0 drwxr-xr-x@   3 rmeira  staff     102 Apr  6 11:02 rpm
````

Make sure you are in the correct directory and proceed with the installation:
````
$ cd /work/google-cloud-sdk/
$ ./install.sh
````
You should see an output similar to the example shown below:
```
$ ./install.sh
Welcome to the Google Cloud SDK!

To help improve the quality of this product, we collect anonymized usage data
and anonymized stacktraces when crashes are encountered; additional information
is available at <https://cloud.google.com/sdk/usage-statistics>. You may choose
to opt out of this collection now (by choosing 'N' at the below prompt), or at
any time in the future by running the following command:

    gcloud config set disable_usage_reporting true

Do you want to help improve the Google Cloud SDK (Y/n)?  Y

Your current Cloud SDK version is: 197.0.0
The latest available version is: 197.0.0

┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                  Components                                                 │
├───────────────┬──────────────────────────────────────────────────────┬──────────────────────────┬───────────┤
│     Status    │                         Name                         │            ID            │    Size   │
├───────────────┼──────────────────────────────────────────────────────┼──────────────────────────┼───────────┤
│ Not Installed │ App Engine Go Extensions                             │ app-engine-go            │ 151.3 MiB │
│ Not Installed │ Cloud Bigtable Command Line Tool                     │ cbt                      │   4.7 MiB │
│ Not Installed │ Cloud Bigtable Emulator                              │ bigtable                 │   3.8 MiB │
│ Not Installed │ Cloud Datalab Command Line Tool                      │ datalab                  │   < 1 MiB │
│ Not Installed │ Cloud Datastore Emulator                             │ cloud-datastore-emulator │  17.9 MiB │
│ Not Installed │ Cloud Datastore Emulator (Legacy)                    │ gcd-emulator             │  38.1 MiB │
│ Not Installed │ Cloud Pub/Sub Emulator                               │ pubsub-emulator          │  33.4 MiB │
│ Not Installed │ Emulator Reverse Proxy                               │ emulator-reverse-proxy   │  14.5 MiB │
│ Not Installed │ Google Container Local Builder                       │ container-builder-local  │   3.7 MiB │
│ Not Installed │ Google Container Registry's Docker credential helper │ docker-credential-gcr    │   2.5 MiB │
│ Not Installed │ gcloud Alpha Commands                                │ alpha                    │   < 1 MiB │
│ Not Installed │ gcloud Beta Commands                                 │ beta                     │   < 1 MiB │
│ Not Installed │ gcloud app Java Extensions                           │ app-engine-java          │ 118.9 MiB │
│ Not Installed │ gcloud app PHP Extensions                            │ app-engine-php           │  21.9 MiB │
│ Not Installed │ gcloud app Python Extensions                         │ app-engine-python        │   6.2 MiB │
│ Not Installed │ gcloud app Python Extensions (Extra Libraries)       │ app-engine-python-extras │  27.8 MiB │
│ Not Installed │ kubectl                                              │ kubectl                  │  12.2 MiB │
│ Installed     │ BigQuery Command Line Tool                           │ bq                       │   < 1 MiB │
│ Installed     │ Cloud SDK Core Libraries                             │ core                     │   7.5 MiB │
│ Installed     │ Cloud Storage Command Line Tool                      │ gsutil                   │   3.4 MiB │
└───────────────┴──────────────────────────────────────────────────────┴──────────────────────────┴───────────┘
To install or remove components at your current SDK version [197.0.0], run:
  $ gcloud components install COMPONENT_ID
  $ gcloud components remove COMPONENT_ID

To update your SDK installation to the latest version [197.0.0], run:
  $ gcloud components update

Modify profile to update your $PATH and enable shell command 
completion?

Do you want to continue (Y/n)?  Y

The Google Cloud SDK installer will now prompt you to update an rc 
file to bring the Google Cloud CLIs into your environment.

Enter a path to an rc file to update, or leave blank to use 
[/Users/rmeira/.bash_profile]:  
No changes necessary for [/Users/rmeira/.bash_profile].

For more information on how to get started, please visit:
  https://cloud.google.com/sdk/docs/quickstarts

```

Using a *new* Terminal window on my  Mac, so that any newly created environment variables are taken into account, proceed as follows:

````
$ cd /work/google-cloud-sdk/
$ ./bin/gcloud init
````

You should see an output similar to the example shown below. Note that the process is interactive. It will demand inputs to complete successfully. I indicate the places where `<<< user input is required`:

```
$ ./bin/gcloud init
Welcome! This command will take you through the configuration of gcloud.

Settings from your current configuration [default] are:
compute:
  region: us-east1
  zone: us-east1-b
core:
  account: rmeira@pivotal.io
  disable_usage_reporting: 'False'
  project: fe-rmeira

Pick configuration to use:
 [1] Re-initialize this configuration [default] with new settings 
 [2] Create a new configuration
Please enter your numeric choice:  1                                      <<< user input is required

Your current configuration has been set to: [default]

You can skip diagnostics next time by using the following flag:
  gcloud init --skip-diagnostics

Network diagnostic detects and fixes local network connection issues.
Checking network connection...done.                                                                                                  
Reachability Check passed.
Network diagnostic (1/1 checks) passed.

Choose the account you would like to use to perform operations for 
this configuration:
 [1] rmeira@pivotal.io
 [2] Log in with a new account
Please enter your numeric choice:  1                                      <<< user input is required 

You are logged in as: [rmeira@pivotal.io].

Pick cloud project to use: 
 [1] fe-rmeira
 [2] pde-greenplum-dev
 [3] Create a new project
Please enter numeric choice or text value:  1                             <<< user input is required

Your current project has been set to: [fe-rmeira].

Do you want to configure a default Compute Region and Zone? (Y/n)?  Y     <<< user input is required

Which Google Compute Engine zone would you like to use as project 
default?
If you do not specify a zone via a command line flag while working 
with Compute Engine resources, the default is assumed.
 [1] us-east1-b
 [2] us-east1-c
 [3] us-east1-d
 [4] us-east4-c
 [5] us-east4-b
 [6] us-east4-a
 [7] us-central1-c
 [8] us-central1-a
 [9] us-central1-f
 [10] us-central1-b
 [11] us-west1-b
 [12] us-west1-c
 [13] us-west1-a
 [14] europe-west4-a
 [15] europe-west4-b
 [16] europe-west4-c
 [17] europe-west1-b
 [18] europe-west1-d
 [19] europe-west1-c
 [20] europe-west3-b
 [21] europe-west3-c
 [22] europe-west3-a
 [23] europe-west2-c
 [24] europe-west2-b
 [25] europe-west2-a
 [26] asia-east1-b
 [27] asia-east1-a
 [28] asia-east1-c
 [29] asia-southeast1-b
 [30] asia-southeast1-a
 [31] asia-northeast1-b
 [32] asia-northeast1-c
 [33] asia-northeast1-a
 [34] asia-south1-c
 [35] asia-south1-b
 [36] asia-south1-a
 [37] australia-southeast1-b
 [38] australia-southeast1-c
 [39] australia-southeast1-a
 [40] southamerica-east1-b
 [41] southamerica-east1-c
 [42] southamerica-east1-a
 [43] northamerica-northeast1-a
 [44] northamerica-northeast1-b
 [45] northamerica-northeast1-c
 [46] Do not set default zone
Please enter numeric choice or text value:  1                             <<< user input is required

Your project default Compute Engine zone has been set to [us-east1-b].
You can change it by running [gcloud config set compute/zone NAME].

Your project default Compute Engine region has been set to [us-east1].
You can change it by running [gcloud config set compute/region NAME].

Your Google Cloud SDK is configured and ready to use!

* Commands that require authentication will use rmeira@pivotal.io by default
* Commands will reference project `fe-rmeira` by default
* Compute Engine commands will use region `us-east1` by default
* Compute Engine commands will use zone `us-east1-b` by default

Run `gcloud help config` to learn how to change individual settings

This gcloud configuration is called [default]. You can create additional configurations if you work with multiple accounts and/or projects.
Run `gcloud topic configurations` to learn more.

Some things to try next:

* Run `gcloud --help` to see the Cloud Platform services you can interact with. And run `gcloud help COMMAND` to get help on any gcloud command.
* Run `gcloud topic -h` to learn about advanced features of the SDK like arg files and output formatting
```

Let's check what do I have installed:
````
$ gcloud components list
````
The expected results should look something like this:
```
$ gcloud components list

Your current Cloud SDK version is: 197.0.0
The latest available version is: 197.0.0

┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                  Components                                                 │
├───────────────┬──────────────────────────────────────────────────────┬──────────────────────────┬───────────┤
│     Status    │                         Name                         │            ID            │    Size   │
├───────────────┼──────────────────────────────────────────────────────┼──────────────────────────┼───────────┤
│ Not Installed │ App Engine Go Extensions                             │ app-engine-go            │ 151.3 MiB │
│ Not Installed │ Cloud Bigtable Command Line Tool                     │ cbt                      │   4.7 MiB │
│ Not Installed │ Cloud Bigtable Emulator                              │ bigtable                 │   3.8 MiB │
│ Not Installed │ Cloud Datalab Command Line Tool                      │ datalab                  │   < 1 MiB │
│ Not Installed │ Cloud Datastore Emulator                             │ cloud-datastore-emulator │  17.9 MiB │
│ Not Installed │ Cloud Datastore Emulator (Legacy)                    │ gcd-emulator             │  38.1 MiB │
│ Not Installed │ Cloud Pub/Sub Emulator                               │ pubsub-emulator          │  33.4 MiB │
│ Not Installed │ Emulator Reverse Proxy                               │ emulator-reverse-proxy   │  14.5 MiB │
│ Not Installed │ Google Container Local Builder                       │ container-builder-local  │   3.7 MiB │
│ Not Installed │ Google Container Registry's Docker credential helper │ docker-credential-gcr    │   2.5 MiB │
│ Not Installed │ gcloud Alpha Commands                                │ alpha                    │   < 1 MiB │
│ Not Installed │ gcloud Beta Commands                                 │ beta                     │   < 1 MiB │
│ Not Installed │ gcloud app Java Extensions                           │ app-engine-java          │ 118.9 MiB │
│ Not Installed │ gcloud app PHP Extensions                            │ app-engine-php           │  21.9 MiB │
│ Not Installed │ gcloud app Python Extensions                         │ app-engine-python        │   6.2 MiB │
│ Not Installed │ gcloud app Python Extensions (Extra Libraries)       │ app-engine-python-extras │  27.8 MiB │
│ Not Installed │ kubectl                                              │ kubectl                  │  12.2 MiB │
│ Installed     │ BigQuery Command Line Tool                           │ bq                       │   < 1 MiB │
│ Installed     │ Cloud SDK Core Libraries                             │ core                     │   7.5 MiB │
│ Installed     │ Cloud Storage Command Line Tool                      │ gsutil                   │   3.4 MiB │
└───────────────┴──────────────────────────────────────────────────────┴──────────────────────────┴───────────┘
To install or remove components at your current SDK version [197.0.0], run:
  $ gcloud components install COMPONENT_ID
  $ gcloud components remove COMPONENT_ID

To update your SDK installation to the latest version [197.0.0], run:
  $ gcloud components update
```

Let's check my configuration by using the following command: `gcloud config list`. 
````
$ gcloud config list
[compute]
region = us-east1
zone = us-east1-b
[core]
account = rmeira@pivotal.io
disable_usage_reporting = False
project = fe-rmeira

Your active configuration is: [default]
````
The resulting values shown above are very important because they define where the VMs and Storage Buckets will be created by default, and by what account and project_id.

### Setup your workstation
We need to set-up environment variables on your workstation (in my case, my Mac) before proceeding with the creation of VMs and storage buckets:

1. Set your project ID:

  ```
  $ export projectid=REPLACE_WITH_YOUR_PROJECT_ID
  ```
  
  Which in my case is `export projectid=fe-rmeira; echo $projectid`

2. Export your preferred compute region and zone:

  ```
  $ export region=us-east1
  $ export zone=us-east1-b
  $ export zone2=us-east1-c
  ```
  For easy copy-&-paste, I used:  `export region=us-east1; export zone=us-east1-b; export zone2=us-east1-c;`  
  
  Why did I pick, zone2 to be us-east1-c? For no other reason than the fact that us-east1-c should be close to us-east1-b - both being located somewhere around Moncks Corner, a town in South Carolina. You can find additional details about GCP regions, zones and the features of the VMs and storage buckets by accessing https://cloud.google.com/compute/docs/regions-zones/
  
3. Configure `gcloud` with a user who is an owner of the project:

  ```
  $ gcloud auth login
  ```
  The command above opens up a browser at `https://cloud.google.com/sdk/auth_success` to authenticate you. 
  You should see the following output on your  Mac terminal: 
  
  ```
  Your browser has been opened to visit:

    https://accounts.google.com/o/oauth2/auth?redirect_uri=http%3A%2F%2Flocalhost%3A8085%2F&prompt=select_account&response_type=code&client_id=3212345640559.apps.googleusercontent.com&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.admin+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcompute+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Faccounts.reauth&access_type=offline


WARNING: \`gcloud auth login\` no longer writes application default credentials.
If you need to use ADC, see:
  gcloud auth application-default --help

You are now logged in as [rmeira@pivotal.io].
Your current project is [fe-rmeira].  You can change this setting by running:
  $ gcloud config set project PROJECT_ID
  ```
  
  and on the browser window that should have automatically opened up, you should see:
  
  ![](./google-sdk-auth.png)
  
  Proceed with the following commands to set gcloud target configuration project, zone and region values:
  ```
  $ gcloud config set project ${projectid}
  $ gcloud config set compute/zone ${zone}
  $ gcloud config set compute/region ${region}
  ```
  For easy copy-&-paste, I used:  `gcloud config set project ${projectid}; gcloud config set compute/zone ${zone}; gcloud config set compute/region ${region}; `  
    
  You should see the following output:
  
  ```
  $ gcloud config set project ${projectid}
  Updated property [core/project].
  $ gcloud config set compute/zone ${zone}
  Updated property [compute/zone].
  $ gcloud config set compute/region ${region}
  Updated property [compute/region].
  ```
     
4. Now we need to create a service account and service key to enable Terraform to execute scripts that will create infrastructure in GCP:

  ```
  $ gcloud iam service-accounts create terraform-bosh
  $ gcloud iam service-accounts keys create /tmp/terraform-bosh.key.json \
      --iam-account terraform-bosh@${projectid}.iam.gserviceaccount.com
  ```
  
  The expected output you should see:
  ```
  $ gcloud iam service-accounts create terraform-bosh
  Created service account [terraform-bosh].
  ```
  and the results from the `gcloud iam service-accounts keys create...` command are:
  ```
  $ gcloud iam service-accounts keys create /tmp/terraform-bosh.key.json --iam-account terraform-bosh@${projectid}.iam.gserviceaccount.com
  created key [172691632345678ddd] of type [json] as [/tmp/terraform-bosh.key.json] for [terraform-bosh@fe-rmeira.iam.gserviceaccount.com]
  ```
  It happened to me, so just to be sure I don't leave you hanging, if you need to delete the terraform-bosh service-account, use the following command:
  ```
  $ gcloud iam service-accounts delete terraform-bosh@fe-rmeira.iam.gserviceaccount.com
You are about to delete service account 
[terraform-bosh@fe-rmeira.iam.gserviceaccount.com].

Do you want to continue (Y/n)?  Y

deleted service account [terraform-bosh@fe-rmeira.iam.gserviceaccount.com]
  ```

5. Now let's grant the new service account (which will be used by Terraform) the editor level access to your project:

  ```
  $ gcloud projects add-iam-policy-binding ${projectid} \
    --member serviceAccount:terraform-bosh@${projectid}.iam.gserviceaccount.com \
    --role roles/editor
  ```
  
  The expected output should be similar to the example below:
  ```
  $ gcloud projects add-iam-policy-binding ${projectid} --member serviceAccount:terraform-bosh@${projectid}.iam.gserviceaccount.com --role roles/editor
bindings:
- members:
  - serviceAccount:service-606020681667@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:service-606020681667@dataflow-service-producer-prod.iam.gserviceaccount.com
  role: roles/dataflow.serviceAgent
- members:
  - serviceAccount:606020681667-compute@developer.gserviceaccount.com
  - serviceAccount:606020681667@cloudservices.gserviceaccount.com
  - serviceAccount:fe-rmeira@appspot.gserviceaccount.com
  - serviceAccount:service-606020681667@containerregistry.iam.gserviceaccount.com
  - serviceAccount:terraform-bosh@fe-rmeira.iam.gserviceaccount.com
  role: roles/editor
- members:
  - user:claurence@pivotal.io
  - user:mrosado@pivotal.io
  - user:rmeira@pivotal.io
  - user:tsavage@pivotal.io
  role: roles/owner
etag: BwVpmCGKBpw=
version: 1
  ```

6. Let's make your service account's key is available in an environment variable to be used by Terraform:

  ```
  $ export GOOGLE_CREDENTIALS=$(cat /tmp/terraform-bosh.key.json)
  ```
  
  You can check the contents of GOOGLE_CREDENTIALS as follows:
  ```
  $ echo $GOOGLE_CREDENTIALS
{ "type": "service_account", "project_id": "fe-rmeira", "private_key_id": "17269656872512345678909a0ddd", 
   "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBK1234567890eznJfMSp6sPr\n3Hum+VwnZ+5FDuwFGU5EbW0bLhnmpK44f0h2w/kSMe3qebZPyuuZ2\n
  3bFTXfx7miBszwvPdgvbqXDRAs+AhoYEsmatR\nQSNFAyDRoJeM+2NPCPSRpK9U\n-----END PRIVATE KEY-----\n",
  "client_email": "terraform-bosh@fe-rmeira.iam.gserviceaccount.com", "client_id": "111780899105249296126", "auth_uri":
  "https://accounts.google.com/o/oauth2/auth", "token_uri": "https://accounts.google.com/o/oauth2/token", "auth_provider_x509_cert_url": 
  "https://www.googleapis.com/oauth2/v1/certs", "client_x509_cert_url": 
  "https://www.googleapis.com/robot/v1/metadata/x509/terraform-bosh%40fe-rmeira.iam.gserviceaccount.com" }
  ```
  
### Now let's create required GCP infrastructure using Terraform

1. Download [main.tf](main.tf) and [concourse.tf](concourse.tf) from this repository. 

* And take a look at each one of these files to make sure they are pointing to the correct GCP region, zones, etc...
* I decided to place both `main.tf` and `concourse.tf` under the same `/work/google-cloud-sdk`

2. In a terminal on my Mac, from the same directory where the two `.tf` files are located, execute `terraform init` and then view the Terraform execution plan to see the resources that will be created:

  ```
  $ terraform init
  ```  
  
  The expected output should be similar to the example shown below:
  ```
  $ terraform init
  
Initializing provider plugins...
- Checking for available provider plugins on https://releases.hashicorp.com...
- Downloading plugin for provider "google" (1.9.0)...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.google: version = "~> 1.9"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
  ```
  
To view the execution plan, use the following command:

  ```
  $ terraform plan -var projectid=${projectid} -var region=${region} -var zone-1=${zone} -var zone-2=${zone2}
  ```
  
  The expected output should be similar to the example shown below:

  ```
  $ terraform plan -var projectid=${projectid} -var region=${region} -var zone-1=${zone} -var zone-2=${zone2}
  
  Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.
------------------------------------------------------------------------
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + google_compute_address.concourse
      id:                                                  <computed>
      address:                                             <computed>
      address_type:                                        "EXTERNAL"
      name:                                                "concourse"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>

  + google_compute_firewall.bosh-bastion
      id:                                                  <computed>
      allow.#:                                             "2"
      allow.1367131964.ports.#:                            "0"
      allow.1367131964.protocol:                           "icmp"
      allow.803338340.ports.#:                             "1"
      allow.803338340.ports.0:                             "22"
      allow.803338340.protocol:                            "tcp"
      destination_ranges.#:                                <computed>
      direction:                                           <computed>
      name:                                                "bosh-bastion-concourse"
      network:                                             "concourse"
      priority:                                            "1000"
      project:                                             <computed>
      self_link:                                           <computed>
      source_ranges.#:                                     <computed>
      target_tags.#:                                       "1"
      target_tags.1860295641:                              "bosh-bastion"

  + google_compute_firewall.bosh-internal
      id:                                                  <computed>
      allow.#:                                             "3"
      allow.1367131964.ports.#:                            "0"
      allow.1367131964.protocol:                           "icmp"
      allow.1486604749.ports.#:                            "0"
      allow.1486604749.protocol:                           "udp"
      allow.3848845357.ports.#:                            "0"
      allow.3848845357.protocol:                           "tcp"
      destination_ranges.#:                                <computed>
      direction:                                           <computed>
      name:                                                "bosh-internal-concourse"
      network:                                             "concourse"
      priority:                                            "1000"
      project:                                             <computed>
      self_link:                                           <computed>
      source_ranges.#:                                     <computed>
      source_tags.#:                                       "1"
      source_tags.2206338310:                              "bosh-internal"
      target_tags.#:                                       "1"
      target_tags.2206338310:                              "bosh-internal"

  + google_compute_firewall.concourse-internal
      id:                                                  <computed>
      allow.#:                                             "3"
      allow.1367131964.ports.#:                            "0"
      allow.1367131964.protocol:                           "icmp"
      allow.1486604749.ports.#:                            "0"
      allow.1486604749.protocol:                           "udp"
      allow.3848845357.ports.#:                            "0"
      allow.3848845357.protocol:                           "tcp"
      destination_ranges.#:                                <computed>
      direction:                                           <computed>
      name:                                                "concourse-internal"
      network:                                             "concourse"
      priority:                                            "1000"
      project:                                             <computed>
      self_link:                                           <computed>
      source_ranges.#:                                     <computed>
      source_tags.#:                                       "2"
      source_tags.1832082793:                              "concourse-internal"
      source_tags.2206338310:                              "bosh-internal"
      target_tags.#:                                       "2"
      target_tags.1832082793:                              "concourse-internal"
      target_tags.2206338310:                              "bosh-internal"

  + google_compute_firewall.concourse-public
      id:                                                  <computed>
      allow.#:                                             "1"
      allow.3387774561.ports.#:                            "4"
      allow.3387774561.ports.0:                            "80"
      allow.3387774561.ports.1:                            "8080"
      allow.3387774561.ports.2:                            "443"
      allow.3387774561.ports.3:                            "4443"
      allow.3387774561.protocol:                           "tcp"
      destination_ranges.#:                                <computed>
      direction:                                           <computed>
      name:                                                "concourse-public"
      network:                                             "concourse"
      priority:                                            "1000"
      project:                                             <computed>
      self_link:                                           <computed>
      source_ranges.#:                                     "1"
      source_ranges.1080289494:                            "0.0.0.0/0"
      target_tags.#:                                       "1"
      target_tags.1288521255:                              "concourse-public"

  + google_compute_forwarding_rule.concourse-http-forwarding-rule
      id:                                                  <computed>
      ip_address:                                          "${google_compute_address.concourse.address}"
      ip_protocol:                                         "TCP"
      load_balancing_scheme:                               "EXTERNAL"
      name:                                                "concourse-http-forwarding-rule"
      port_range:                                          "80-80"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>
      subnetwork:                                          <computed>
      target:                                              "${google_compute_target_pool.concourse-target-pool.self_link}"

  + google_compute_forwarding_rule.concourse-https-forwarding-rule
      id:                                                  <computed>
      ip_address:                                          "${google_compute_address.concourse.address}"
      ip_protocol:                                         "TCP"
      load_balancing_scheme:                               "EXTERNAL"
      name:                                                "concourse-https-forwarding-rule"
      port_range:                                          "443-443"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>
      subnetwork:                                          <computed>
      target:                                              "${google_compute_target_pool.concourse-target-pool.self_link}"

  + google_compute_instance.bosh-bastion
      id:                                                  <computed>
      boot_disk.#:                                         "1"
      boot_disk.0.auto_delete:                             "true"
      boot_disk.0.device_name:                             <computed>
      boot_disk.0.disk_encryption_key_sha256:              <computed>
      boot_disk.0.initialize_params.#:                     "1"
      boot_disk.0.initialize_params.0.image:               "ubuntu-1404-trusty-v20180122"
      boot_disk.0.initialize_params.0.size:                <computed>
      boot_disk.0.initialize_params.0.type:                <computed>
      can_ip_forward:                                      "false"
      cpu_platform:                                        <computed>
      create_timeout:                                      "4"
      deletion_protection:                                 "false"
      guest_accelerator.#:                                 <computed>
      instance_id:                                         <computed>
      label_fingerprint:                                   <computed>
      machine_type:                                        "n1-standard-1"
      metadata_fingerprint:                                <computed>
      metadata_startup_script:                             "#!/bin/bash\napt-get update -y\napt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3\ngem install bosh_cli\ncurl -o /tmp/cf.tgz https://s3.amazonaws.com/go-cli/releases/v6.20.0/cf-cli_6.20.0_linux_x86-64.tgz\ntar -zxvf /tmp/cf.tgz && mv cf /usr/bin/cf && chmod +x /usr/bin/cf\ncurl -o /usr/bin/bosh-init https://s3.amazonaws.com/bosh-init-artifacts/bosh-init-0.0.96-linux-amd64\nchmod +x /usr/bin/bosh-init\n"
      name:                                                "bosh-bastion-concourse"
      network_interface.#:                                 "1"
      network_interface.0.access_config.#:                 "1"
      network_interface.0.access_config.0.assigned_nat_ip: <computed>
      network_interface.0.access_config.0.nat_ip:          <computed>
      network_interface.0.address:                         <computed>
      network_interface.0.name:                            <computed>
      network_interface.0.network_ip:                      <computed>
      network_interface.0.subnetwork:                      "bosh-concourse-us-east1"
      network_interface.0.subnetwork_project:              <computed>
      project:                                             <computed>
      scheduling.#:                                        <computed>
      self_link:                                           <computed>
      service_account.#:                                   "1"
      service_account.0.email:                             <computed>
      service_account.0.scopes.#:                          "1"
      service_account.0.scopes.1733087937:                 "https://www.googleapis.com/auth/cloud-platform"
      tags.#:                                              "2"
      tags.1860295641:                                     "bosh-bastion"
      tags.2206338310:                                     "bosh-internal"
      tags_fingerprint:                                    <computed>
      zone:                                                "us-east1-b"

  + google_compute_network.network
      id:                                                  <computed>
      auto_create_subnetworks:                             "true"
      gateway_ipv4:                                        <computed>
      name:                                                "concourse"
      project:                                             <computed>
      routing_mode:                                        <computed>
      self_link:                                           <computed>

  + google_compute_subnetwork.bosh-subnet-1
      id:                                                  <computed>
      fingerprint:                                         <computed>
      gateway_address:                                     <computed>
      ip_cidr_range:                                       "10.0.0.0/24"
      name:                                                "bosh-concourse-us-east1"
      network:                                             "${google_compute_network.network.self_link}"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>

  + google_compute_subnetwork.concourse-public-subnet-1
      id:                                                  <computed>
      fingerprint:                                         <computed>
      gateway_address:                                     <computed>
      ip_cidr_range:                                       "10.120.0.0/16"
      name:                                                "concourse-public-us-east1-1"
      network:                                             "${google_compute_network.network.self_link}"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>

  + google_compute_subnetwork.concourse-public-subnet-2
      id:                                                  <computed>
      fingerprint:                                         <computed>
      gateway_address:                                     <computed>
      ip_cidr_range:                                       "10.121.0.0/16"
      name:                                                "concourse-public-us-east1-2"
      network:                                             "${google_compute_network.network.self_link}"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>

  + google_compute_target_pool.concourse-target-pool
      id:                                                  <computed>
      instances.#:                                         <computed>
      name:                                                "concourse-target-pool"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>
      session_affinity:                                    "NONE"


Plan: 13 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.

  ```

3. We can now execute the Terraform scripts to create the GCP resources:

  ```
  terraform apply -var projectid=${projectid} -var region=${region} -var zone-1=${zone} -var zone-2=${zone2}
  ```
  
  You should see an output similar the example shown below:
  
  ```
  $ terraform apply -var projectid=${projectid} -var region=${region} -var zone-1=${zone} -var zone-2=${zone2}

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + google_compute_address.concourse
      id:                                                  <computed>
      address:                                             <computed>
      address_type:                                        "EXTERNAL"
      name:                                                "concourse"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>

  + google_compute_firewall.bosh-bastion
      id:                                                  <computed>
      allow.#:                                             "2"
      allow.1367131964.ports.#:                            "0"
      allow.1367131964.protocol:                           "icmp"
      allow.803338340.ports.#:                             "1"
      allow.803338340.ports.0:                             "22"
      allow.803338340.protocol:                            "tcp"
      destination_ranges.#:                                <computed>
      direction:                                           <computed>
      name:                                                "bosh-bastion-concourse"
      network:                                             "concourse"
      priority:                                            "1000"
      project:                                             <computed>
      self_link:                                           <computed>
      source_ranges.#:                                     <computed>
      target_tags.#:                                       "1"
      target_tags.1860295641:                              "bosh-bastion"

  + google_compute_firewall.bosh-internal
      id:                                                  <computed>
      allow.#:                                             "3"
      allow.1367131964.ports.#:                            "0"
      allow.1367131964.protocol:                           "icmp"
      allow.1486604749.ports.#:                            "0"
      allow.1486604749.protocol:                           "udp"
      allow.3848845357.ports.#:                            "0"
      allow.3848845357.protocol:                           "tcp"
      destination_ranges.#:                                <computed>
      direction:                                           <computed>
      name:                                                "bosh-internal-concourse"
      network:                                             "concourse"
      priority:                                            "1000"
      project:                                             <computed>
      self_link:                                           <computed>
      source_ranges.#:                                     <computed>
      source_tags.#:                                       "1"
      source_tags.2206338310:                              "bosh-internal"
      target_tags.#:                                       "1"
      target_tags.2206338310:                              "bosh-internal"

  + google_compute_firewall.concourse-internal
      id:                                                  <computed>
      allow.#:                                             "3"
      allow.1367131964.ports.#:                            "0"
      allow.1367131964.protocol:                           "icmp"
      allow.1486604749.ports.#:                            "0"
      allow.1486604749.protocol:                           "udp"
      allow.3848845357.ports.#:                            "0"
      allow.3848845357.protocol:                           "tcp"
      destination_ranges.#:                                <computed>
      direction:                                           <computed>
      name:                                                "concourse-internal"
      network:                                             "concourse"
      priority:                                            "1000"
      project:                                             <computed>
      self_link:                                           <computed>
      source_ranges.#:                                     <computed>
      source_tags.#:                                       "2"
      source_tags.1832082793:                              "concourse-internal"
      source_tags.2206338310:                              "bosh-internal"
      target_tags.#:                                       "2"
      target_tags.1832082793:                              "concourse-internal"
      target_tags.2206338310:                              "bosh-internal"

  + google_compute_firewall.concourse-public
      id:                                                  <computed>
      allow.#:                                             "1"
      allow.3387774561.ports.#:                            "4"
      allow.3387774561.ports.0:                            "80"
      allow.3387774561.ports.1:                            "8080"
      allow.3387774561.ports.2:                            "443"
      allow.3387774561.ports.3:                            "4443"
      allow.3387774561.protocol:                           "tcp"
      destination_ranges.#:                                <computed>
      direction:                                           <computed>
      name:                                                "concourse-public"
      network:                                             "concourse"
      priority:                                            "1000"
      project:                                             <computed>
      self_link:                                           <computed>
      source_ranges.#:                                     "1"
      source_ranges.1080289494:                            "0.0.0.0/0"
      target_tags.#:                                       "1"
      target_tags.1288521255:                              "concourse-public"

  + google_compute_forwarding_rule.concourse-http-forwarding-rule
      id:                                                  <computed>
      ip_address:                                          "${google_compute_address.concourse.address}"
      ip_protocol:                                         "TCP"
      load_balancing_scheme:                               "EXTERNAL"
      name:                                                "concourse-http-forwarding-rule"
      port_range:                                          "80-80"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>
      subnetwork:                                          <computed>
      target:                                              "${google_compute_target_pool.concourse-target-pool.self_link}"

  + google_compute_forwarding_rule.concourse-https-forwarding-rule
      id:                                                  <computed>
      ip_address:                                          "${google_compute_address.concourse.address}"
      ip_protocol:                                         "TCP"
      load_balancing_scheme:                               "EXTERNAL"
      name:                                                "concourse-https-forwarding-rule"
      port_range:                                          "443-443"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>
      subnetwork:                                          <computed>
      target:                                              "${google_compute_target_pool.concourse-target-pool.self_link}"

  + google_compute_instance.bosh-bastion
      id:                                                  <computed>
      boot_disk.#:                                         "1"
      boot_disk.0.auto_delete:                             "true"
      boot_disk.0.device_name:                             <computed>
      boot_disk.0.disk_encryption_key_sha256:              <computed>
      boot_disk.0.initialize_params.#:                     "1"
      boot_disk.0.initialize_params.0.image:               "ubuntu-1404-trusty-v20180122"
      boot_disk.0.initialize_params.0.size:                <computed>
      boot_disk.0.initialize_params.0.type:                <computed>
      can_ip_forward:                                      "false"
      cpu_platform:                                        <computed>
      create_timeout:                                      "4"
      deletion_protection:                                 "false"
      guest_accelerator.#:                                 <computed>
      instance_id:                                         <computed>
      label_fingerprint:                                   <computed>
      machine_type:                                        "n1-standard-1"
      metadata_fingerprint:                                <computed>
      metadata_startup_script:                             "#!/bin/bash\napt-get update -y\napt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3\ngem install bosh_cli\ncurl -o /tmp/cf.tgz https://s3.amazonaws.com/go-cli/releases/v6.20.0/cf-cli_6.20.0_linux_x86-64.tgz\ntar -zxvf /tmp/cf.tgz && mv cf /usr/bin/cf && chmod +x /usr/bin/cf\ncurl -o /usr/bin/bosh-init https://s3.amazonaws.com/bosh-init-artifacts/bosh-init-0.0.96-linux-amd64\nchmod +x /usr/bin/bosh-init\n"
      name:                                                "bosh-bastion-concourse"
      network_interface.#:                                 "1"
      network_interface.0.access_config.#:                 "1"
      network_interface.0.access_config.0.assigned_nat_ip: <computed>
      network_interface.0.access_config.0.nat_ip:          <computed>
      network_interface.0.address:                         <computed>
      network_interface.0.name:                            <computed>
      network_interface.0.network_ip:                      <computed>
      network_interface.0.subnetwork:                      "bosh-concourse-us-east1"
      network_interface.0.subnetwork_project:              <computed>
      project:                                             <computed>
      scheduling.#:                                        <computed>
      self_link:                                           <computed>
      service_account.#:                                   "1"
      service_account.0.email:                             <computed>
      service_account.0.scopes.#:                          "1"
      service_account.0.scopes.1733087937:                 "https://www.googleapis.com/auth/cloud-platform"
      tags.#:                                              "2"
      tags.1860295641:                                     "bosh-bastion"
      tags.2206338310:                                     "bosh-internal"
      tags_fingerprint:                                    <computed>
      zone:                                                "us-east1-b"

  + google_compute_network.network
      id:                                                  <computed>
      auto_create_subnetworks:                             "true"
      gateway_ipv4:                                        <computed>
      name:                                                "concourse"
      project:                                             <computed>
      routing_mode:                                        <computed>
      self_link:                                           <computed>

  + google_compute_subnetwork.bosh-subnet-1
      id:                                                  <computed>
      fingerprint:                                         <computed>
      gateway_address:                                     <computed>
      ip_cidr_range:                                       "10.0.0.0/24"
      name:                                                "bosh-concourse-us-east1"
      network:                                             "${google_compute_network.network.self_link}"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>

  + google_compute_subnetwork.concourse-public-subnet-1
      id:                                                  <computed>
      fingerprint:                                         <computed>
      gateway_address:                                     <computed>
      ip_cidr_range:                                       "10.120.0.0/16"
      name:                                                "concourse-public-us-east1-1"
      network:                                             "${google_compute_network.network.self_link}"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>

  + google_compute_subnetwork.concourse-public-subnet-2
      id:                                                  <computed>
      fingerprint:                                         <computed>
      gateway_address:                                     <computed>
      ip_cidr_range:                                       "10.121.0.0/16"
      name:                                                "concourse-public-us-east1-2"
      network:                                             "${google_compute_network.network.self_link}"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>

  + google_compute_target_pool.concourse-target-pool
      id:                                                  <computed>
      instances.#:                                         <computed>
      name:                                                "concourse-target-pool"
      project:                                             <computed>
      region:                                              <computed>
      self_link:                                           <computed>
      session_affinity:                                    "NONE"


Plan: 13 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_address.concourse: Creating...
  address:      "" => "<computed>"
  address_type: "" => "EXTERNAL"
  name:         "" => "concourse"
  project:      "" => "<computed>"
  region:       "" => "<computed>"
  self_link:    "" => "<computed>"
google_compute_target_pool.concourse-target-pool: Creating...
  instances.#:      "" => "<computed>"
  name:             "" => "concourse-target-pool"
  project:          "" => "<computed>"
  region:           "" => "<computed>"
  self_link:        "" => "<computed>"
  session_affinity: "" => "NONE"
google_compute_network.network: Creating...
  auto_create_subnetworks: "" => "true"
  gateway_ipv4:            "" => "<computed>"
  name:                    "" => "concourse"
  project:                 "" => "<computed>"
  routing_mode:            "" => "<computed>"
  self_link:               "" => "<computed>"
google_compute_network.network: Still creating... (10s elapsed)
google_compute_address.concourse: Still creating... (10s elapsed)
google_compute_target_pool.concourse-target-pool: Still creating... (10s elapsed)
google_compute_address.concourse: Creation complete after 13s (ID: projects/fe-rmeira/regions/us-east1/addresses/concourse)
google_compute_target_pool.concourse-target-pool: Creation complete after 13s (ID: concourse-target-pool)
google_compute_forwarding_rule.concourse-https-forwarding-rule: Creating...
  ip_address:            "" => "35.196.40.28"
  ip_protocol:           "" => "TCP"
  load_balancing_scheme: "" => "EXTERNAL"
  name:                  "" => "concourse-https-forwarding-rule"
  port_range:            "" => "443-443"
  project:               "" => "<computed>"
  region:                "" => "<computed>"
  self_link:             "" => "<computed>"
  subnetwork:            "" => "<computed>"
  target:                "" => "https://www.googleapis.com/compute/v1/projects/fe-rmeira/regions/us-east1/targetPools/concourse-target-pool"
google_compute_forwarding_rule.concourse-http-forwarding-rule: Creating...
  ip_address:            "" => "35.196.40.28"
  ip_protocol:           "" => "TCP"
  load_balancing_scheme: "" => "EXTERNAL"
  name:                  "" => "concourse-http-forwarding-rule"
  port_range:            "" => "80-80"
  project:               "" => "<computed>"
  region:                "" => "<computed>"
  self_link:             "" => "<computed>"
  subnetwork:            "" => "<computed>"
  target:                "" => "https://www.googleapis.com/compute/v1/projects/fe-rmeira/regions/us-east1/targetPools/concourse-target-pool"
google_compute_network.network: Still creating... (20s elapsed)
google_compute_forwarding_rule.concourse-https-forwarding-rule: Still creating... (10s elapsed)
google_compute_forwarding_rule.concourse-http-forwarding-rule: Still creating... (10s elapsed)
google_compute_forwarding_rule.concourse-https-forwarding-rule: Creation complete after 12s (ID: concourse-https-forwarding-rule)
google_compute_forwarding_rule.concourse-http-forwarding-rule: Creation complete after 13s (ID: concourse-http-forwarding-rule)
google_compute_network.network: Still creating... (30s elapsed)
google_compute_network.network: Creation complete after 38s (ID: concourse)
google_compute_subnetwork.concourse-public-subnet-1: Creating...
  fingerprint:     "" => "<computed>"
  gateway_address: "" => "<computed>"
  ip_cidr_range:   "" => "10.120.0.0/16"
  name:            "" => "concourse-public-us-east1-1"
  network:         "" => "https://www.googleapis.com/compute/v1/projects/fe-rmeira/global/networks/concourse"
  project:         "" => "<computed>"
  region:          "" => "<computed>"
  self_link:       "" => "<computed>"
google_compute_subnetwork.bosh-subnet-1: Creating...
  fingerprint:     "" => "<computed>"
  gateway_address: "" => "<computed>"
  ip_cidr_range:   "" => "10.0.0.0/24"
  name:            "" => "bosh-concourse-us-east1"
  network:         "" => "https://www.googleapis.com/compute/v1/projects/fe-rmeira/global/networks/concourse"
  project:         "" => "<computed>"
  region:          "" => "<computed>"
  self_link:       "" => "<computed>"
google_compute_firewall.concourse-internal: Creating...
  allow.#:                   "" => "3"
  allow.1367131964.ports.#:  "" => "0"
  allow.1367131964.protocol: "" => "icmp"
  allow.1486604749.ports.#:  "" => "0"
  allow.1486604749.protocol: "" => "udp"
  allow.3848845357.ports.#:  "" => "0"
  allow.3848845357.protocol: "" => "tcp"
  destination_ranges.#:      "" => "<computed>"
  direction:                 "" => "<computed>"
  name:                      "" => "concourse-internal"
  network:                   "" => "concourse"
  priority:                  "" => "1000"
  project:                   "" => "<computed>"
  self_link:                 "" => "<computed>"
  source_ranges.#:           "" => "<computed>"
  source_tags.#:             "" => "2"
  source_tags.1832082793:    "" => "concourse-internal"
  source_tags.2206338310:    "" => "bosh-internal"
  target_tags.#:             "" => "2"
  target_tags.1832082793:    "" => "concourse-internal"
  target_tags.2206338310:    "" => "bosh-internal"
google_compute_subnetwork.concourse-public-subnet-2: Creating...
  fingerprint:     "" => "<computed>"
  gateway_address: "" => "<computed>"
  ip_cidr_range:   "" => "10.121.0.0/16"
  name:            "" => "concourse-public-us-east1-2"
  network:         "" => "https://www.googleapis.com/compute/v1/projects/fe-rmeira/global/networks/concourse"
  project:         "" => "<computed>"
  region:          "" => "<computed>"
  self_link:       "" => "<computed>"
google_compute_firewall.concourse-public: Creating...
  allow.#:                   "" => "1"
  allow.3387774561.ports.#:  "" => "4"
  allow.3387774561.ports.0:  "" => "80"
  allow.3387774561.ports.1:  "" => "8080"
  allow.3387774561.ports.2:  "" => "443"
  allow.3387774561.ports.3:  "" => "4443"
  allow.3387774561.protocol: "" => "tcp"
  destination_ranges.#:      "" => "<computed>"
  direction:                 "" => "<computed>"
  name:                      "" => "concourse-public"
  network:                   "" => "concourse"
  priority:                  "" => "1000"
  project:                   "" => "<computed>"
  self_link:                 "" => "<computed>"
  source_ranges.#:           "" => "1"
  source_ranges.1080289494:  "" => "0.0.0.0/0"
  target_tags.#:             "" => "1"
  target_tags.1288521255:    "" => "concourse-public"
google_compute_firewall.bosh-bastion: Creating...
  allow.#:                   "" => "2"
  allow.1367131964.ports.#:  "" => "0"
  allow.1367131964.protocol: "" => "icmp"
  allow.803338340.ports.#:   "" => "1"
  allow.803338340.ports.0:   "" => "22"
  allow.803338340.protocol:  "" => "tcp"
  destination_ranges.#:      "" => "<computed>"
  direction:                 "" => "<computed>"
  name:                      "" => "bosh-bastion-concourse"
  network:                   "" => "concourse"
  priority:                  "" => "1000"
  project:                   "" => "<computed>"
  self_link:                 "" => "<computed>"
  source_ranges.#:           "" => "<computed>"
  target_tags.#:             "" => "1"
  target_tags.1860295641:    "" => "bosh-bastion"
google_compute_firewall.bosh-internal: Creating...
  allow.#:                   "" => "3"
  allow.1367131964.ports.#:  "" => "0"
  allow.1367131964.protocol: "" => "icmp"
  allow.1486604749.ports.#:  "" => "0"
  allow.1486604749.protocol: "" => "udp"
  allow.3848845357.ports.#:  "" => "0"
  allow.3848845357.protocol: "" => "tcp"
  destination_ranges.#:      "" => "<computed>"
  direction:                 "" => "<computed>"
  name:                      "" => "bosh-internal-concourse"
  network:                   "" => "concourse"
  priority:                  "" => "1000"
  project:                   "" => "<computed>"
  self_link:                 "" => "<computed>"
  source_ranges.#:           "" => "<computed>"
  source_tags.#:             "" => "1"
  source_tags.2206338310:    "" => "bosh-internal"
  target_tags.#:             "" => "1"
  target_tags.2206338310:    "" => "bosh-internal"
google_compute_subnetwork.bosh-subnet-1: Still creating... (10s elapsed)
google_compute_firewall.concourse-public: Still creating... (10s elapsed)
google_compute_subnetwork.concourse-public-subnet-1: Still creating... (10s elapsed)
google_compute_firewall.concourse-internal: Still creating... (10s elapsed)
google_compute_subnetwork.concourse-public-subnet-2: Still creating... (10s elapsed)
google_compute_firewall.bosh-bastion: Still creating... (10s elapsed)
google_compute_firewall.bosh-internal: Still creating... (10s elapsed)
google_compute_firewall.concourse-internal: Creation complete after 11s (ID: concourse-internal)
google_compute_firewall.bosh-bastion: Creation complete after 13s (ID: bosh-bastion-concourse)
google_compute_firewall.concourse-public: Creation complete after 13s (ID: concourse-public)
google_compute_firewall.bosh-internal: Creation complete after 14s (ID: bosh-internal-concourse)
google_compute_subnetwork.concourse-public-subnet-1: Creation complete after 17s (ID: us-east1/concourse-public-us-east1-1)
google_compute_subnetwork.concourse-public-subnet-2: Creation complete after 18s (ID: us-east1/concourse-public-us-east1-2)
google_compute_subnetwork.bosh-subnet-1: Creation complete after 18s (ID: us-east1/bosh-concourse-us-east1)
google_compute_instance.bosh-bastion: Creating...
  boot_disk.#:                                         "" => "1"
  boot_disk.0.auto_delete:                             "" => "true"
  boot_disk.0.device_name:                             "" => "<computed>"
  boot_disk.0.disk_encryption_key_sha256:              "" => "<computed>"
  boot_disk.0.initialize_params.#:                     "" => "1"
  boot_disk.0.initialize_params.0.image:               "" => "ubuntu-1404-trusty-v20180122"
  boot_disk.0.initialize_params.0.size:                "" => "<computed>"
  boot_disk.0.initialize_params.0.type:                "" => "<computed>"
  can_ip_forward:                                      "" => "false"
  cpu_platform:                                        "" => "<computed>"
  create_timeout:                                      "" => "4"
  deletion_protection:                                 "" => "false"
  guest_accelerator.#:                                 "" => "<computed>"
  instance_id:                                         "" => "<computed>"
  label_fingerprint:                                   "" => "<computed>"
  machine_type:                                        "" => "n1-standard-1"
  metadata_fingerprint:                                "" => "<computed>"
  metadata_startup_script:                             "" => "#!/bin/bash\napt-get update -y\napt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3\ngem install bosh_cli\ncurl -o /tmp/cf.tgz https://s3.amazonaws.com/go-cli/releases/v6.20.0/cf-cli_6.20.0_linux_x86-64.tgz\ntar -zxvf /tmp/cf.tgz && mv cf /usr/bin/cf && chmod +x /usr/bin/cf\ncurl -o /usr/bin/bosh-init https://s3.amazonaws.com/bosh-init-artifacts/bosh-init-0.0.96-linux-amd64\nchmod +x /usr/bin/bosh-init\n"
  name:                                                "" => "bosh-bastion-concourse"
  network_interface.#:                                 "" => "1"
  network_interface.0.access_config.#:                 "" => "1"
  network_interface.0.access_config.0.assigned_nat_ip: "" => "<computed>"
  network_interface.0.access_config.0.nat_ip:          "" => "<computed>"
  network_interface.0.address:                         "" => "<computed>"
  network_interface.0.name:                            "" => "<computed>"
  network_interface.0.network_ip:                      "" => "<computed>"
  network_interface.0.subnetwork:                      "" => "bosh-concourse-us-east1"
  network_interface.0.subnetwork_project:              "" => "<computed>"
  project:                                             "" => "<computed>"
  scheduling.#:                                        "" => "<computed>"
  self_link:                                           "" => "<computed>"
  service_account.#:                                   "" => "1"
  service_account.0.email:                             "" => "<computed>"
  service_account.0.scopes.#:                          "" => "1"
  service_account.0.scopes.1733087937:                 "" => "https://www.googleapis.com/auth/cloud-platform"
  tags.#:                                              "" => "2"
  tags.1860295641:                                     "" => "bosh-bastion"
  tags.2206338310:                                     "" => "bosh-internal"
  tags_fingerprint:                                    "" => "<computed>"
  zone:                                                "" => "us-east1-b"
google_compute_instance.bosh-bastion: Still creating... (10s elapsed)
google_compute_instance.bosh-bastion: Creation complete after 12s (ID: bosh-bastion-concourse)

Apply complete! Resources: 13 added, 0 changed, 0 destroyed.
  ```

# Recap of what just happened

At this point Terraform has successfully created a `bosh-bastion-concourse` VM in GCE.

![](./bosh-bastion-created.png)

The `bosh-bastion-concourse` is a Ubuntu 14.04 VM where the following script, contained in `main.tf`, was executed.

```
#!/bin/bash
apt-get update -y
apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3
gem install bosh_cli
curl -o /tmp/cf.tgz https://s3.amazonaws.com/go-cli/releases/v6.20.0/cf-cli_6.20.0_linux_x86-64.tgz
tar -zxvf /tmp/cf.tgz && mv cf /usr/bin/cf && chmod +x /usr/bin/cf
curl -o /usr/bin/bosh-init https://s3.amazonaws.com/bosh-init-artifacts/bosh-init-0.0.96-linux-amd64
chmod +x /usr/bin/bosh-init
```

Consequently, the `bosh-bastion-concourse` VM has the `Bosh CLI`, the `CF CLI`, and `Bosh-Init` installed.
This `bosh-bastion-concourse` VM is a 1vCPU 3.75GB RAM machine with 10GB of persistent SSD attached.

Looking across the GCP Console, we can see that Terraform was responsible for the creation of:

- An ephemeral external IP address 35.196.2.112 and a primary IP 10.0.0.2 for the `bosh-bastion-concourse` VM
- A `bosh-concourse-us-east1    ` subnet with CIDR	 10.0.0.0/24 and Gateway IP   10.0.0.1 in the us-east1 region
  This is the subnet that will be used by the Bosh Director nad its workers.
- A `concourse-public-us-east1-1` subnet with CIDR 10.120.0.0/16 and Gateway IP 10.120.0.1 in the us-east1 region
- A `concourse-public-us-east1-2` subnet with CIDR 10.121.0.0/16 and Gateway IP 10.121.0.1 in the us-east1 region
- A `concourse                  ` subnet with CIDR 10.142.0.0/20 and Gateway IP 10.142.0.1 in the us-east1 region
- Another 15 `concourse         ` subnets with CIDRs 10.x.0.0/20 and respective Gateways in the other regions
  These 15 subnets were probably created because the `concourse` VPC was created using Subnet creation mode set to `Auto subnets`
- Firewall rules that allow SSH (TCP port 22) access to the `bosh-bastion-concourse` VM
- Firewall rules that allow communication through ports 80, 8080, 443 and 4443 to the Concourse subnets
- A `concourse-target-pool` load-balancer with an external IP address frontend at 35.196.40.28 open to ports 80 and 443










### Deploy a BOSH Director

1. SSH to the bastion VM you created in the previous step. All SSH commands after this should be run from the VM:

  ```
  gcloud compute ssh bosh-bastion-concourse
  ```

2. Configure `gcloud` to use the correct zone, region, and project:

  ```
  zone=$(curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/zone)
  export zone=${zone##*/}
  export region=${zone%-*}
  gcloud config set compute/zone ${zone}
  gcloud config set compute/region ${region}
  export project_id=`curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/project/project-id`
  ```

3. Explicitly set your secondary zone:

  ```
  export zone2=us-east1-c
  ```

4. Create a **password-less** SSH key:

  ```
  ssh-keygen -t rsa -f ~/.ssh/bosh -C bosh
  ```

5. Run this `export` command to set the full path of the SSH private key you created earlier:

  ```
  export ssh_key_path=$HOME/.ssh/bosh
  ```

5. Navigate to your [project's web console](https://console.cloud.google.com/compute/metadata/sshKeys) and add the new SSH public key by pasting the contents of ~/.ssh/bosh.pub:

  ![](../img/add-ssh.png)

  > **Important:** The username field should auto-populate the value `bosh` after you paste the public key. If it does not, be sure there are no newlines or carriage returns being pasted; the value you paste should be a single line.


6. Confirm that `bosh-init` is installed by querying its version:

  ```
  bosh-init -v
  ```

7. Create and `cd` to a directory:

  ```
  mkdir google-bosh-director
  cd google-bosh-director
  ```

8. Use `vim` or `vi` or `nano` to create a BOSH Director deployment manifest named `manifest.yml.erb`:

  ```
  ---
  <%
  ['region', 'project_id', 'zone', 'ssh_key_path'].each do |val|
    if ENV[val].nil? || ENV[val].empty?
      raise "Missing environment variable: #{val}"
    end
  end

  region = ENV['region']
  project_id = ENV['project_id']
  zone = ENV['zone']
  ssh_key_path = ENV['ssh_key_path']
  %>
  name: bosh

  releases:
    - name: bosh
      url: https://bosh.io/d/github.com/cloudfoundry/bosh?v=260.1
      sha1: 7fb8e99e28b67df6604e97ef061c5425460518d3
    - name: bosh-google-cpi
      url: https://bosh.io/d/github.com/cloudfoundry-incubator/bosh-google-cpi-release?v=25.6.2
      sha1: b4865397d867655fdcc112bc5a7f9a5025cdf311

  resource_pools:
    - name: vms
      network: private
      stemcell:
        url: https://bosh.io/d/stemcells/bosh-google-kvm-ubuntu-trusty-go_agent?v=3312.12
        sha1: 3a2c407be6c1b3d04bb292ceb5007159100c85d7
      cloud_properties:
        zone: <%=zone %>
        machine_type: n1-standard-4
        root_disk_size_gb: 40
        root_disk_type: pd-standard
        service_scopes:
          - compute
          - devstorage.full_control

  disk_pools:
    - name: disks
      disk_size: 32_768
      cloud_properties:
        type: pd-standard

  networks:
    - name: vip
      type: vip
    - name: private
      type: manual
      subnets:
      - range: 10.0.0.0/29
        gateway: 10.0.0.1
        static: [10.0.0.3-10.0.0.7]
        cloud_properties:
          network_name: concourse
          subnetwork_name: bosh-concourse-<%=region %>
          ephemeral_external_ip: true
          tags:
            - bosh-internal

  jobs:
    - name: bosh
      instances: 1

      templates:
        - name: nats
          release: bosh
        - name: postgres
          release: bosh
        - name: powerdns
          release: bosh
        - name: blobstore
          release: bosh
        - name: director
          release: bosh
        - name: health_monitor
          release: bosh
        - name: google_cpi
          release: bosh-google-cpi

      resource_pool: vms
      persistent_disk_pool: disks

      networks:
        - name: private
          static_ips: [10.0.0.6]
          default:
            - dns
            - gateway

      properties:
        nats:
          address: 127.0.0.1
          user: nats
          password: nats-password

        postgres: &db
          listen_address: 127.0.0.1
          host: 127.0.0.1
          user: postgres
          password: postgres-password
          database: bosh
          adapter: postgres

        dns:
          address: 10.0.0.6
          domain_name: microbosh
          db: *db
          recursor: 169.254.169.254

        blobstore:
          address: 10.0.0.6
          port: 25250
          provider: dav
          director:
            user: director
            password: director-password
          agent:
            user: agent
            password: agent-password

        director:
          address: 127.0.0.1
          name: micro-google
          db: *db
          cpi_job: google_cpi
          user_management:
            provider: local
            local:
              users:
                - name: admin
                  password: admin
                - name: hm
                  password: hm-password
        hm:
          director_account:
            user: hm
            password: hm-password
          resurrector_enabled: true

        google: &google_properties
          project: <%=project_id %>

        agent:
          mbus: nats://nats:nats-password@10.0.0.6:4222
          ntp: *ntp
          blobstore:
             options:
               endpoint: http://10.0.0.6:25250
               user: agent
               password: agent-password

        ntp: &ntp
          - 169.254.169.254

  cloud_provider:
    template:
      name: google_cpi
      release: bosh-google-cpi

    ssh_tunnel:
      host: 10.0.0.6
      port: 22
      user: bosh
      private_key: <%=ssh_key_path %>

    mbus: https://mbus:mbus-password@10.0.0.6:6868

    properties:
      google: *google_properties
      agent: {mbus: "https://mbus:mbus-password@0.0.0.0:6868"}
      blobstore: {provider: local, path: /var/vcap/micro_bosh/data/cache}
      ntp: *ntp
  ```

9. Fill in the template values of the manifest with your environment variables:
  ```
  erb manifest.yml.erb > manifest.yml
  ```

10. Deploy the new manifest to create a BOSH Director:

  ```
  bosh-init deploy manifest.yml
  ```

11. Target your BOSH environment:

  ```
  bosh target 10.0.0.6
  ```

Your username is `admin` and password is `admin`.

### Deploy Concourse
Complete the following steps from your bastion instance.

1. Upload the required [Google BOSH Stemcell](http://bosh.io/docs/stemcell.html):

  ```
  bosh upload stemcell https://bosh.io/d/stemcells/bosh-google-kvm-ubuntu-trusty-go_agent?v=3263.8
  ```

2. Upload the required [BOSH Releases](http://bosh.io/docs/release.html):

  ```
  bosh upload release https://bosh.io/d/github.com/concourse/concourse?v=2.5.0
  bosh upload release https://bosh.io/d/github.com/cloudfoundry/garden-runc-release?v=1.0.3
  ```

3. Download the [cloud-config.yml](cloud-config.yml) manifest file.

4. Download the [concourse.yml](concourse.yml) manifest file and set a few environment variables:

  ```
  export external_ip=`gcloud compute addresses describe concourse | grep ^address: | cut -f2 -d' '`
  export director_uuid=`bosh status --uuid 2>/dev/null`
  ```
  
  - Take note of the External_IP address. You will need it to connect to Concourse via the [Fly CLI](https://concourse-ci.org/fly-cli.html).
  ```
  echo $external_ip
  ```

5. Choose unique passwords for internal services and ATC and export them
   ```
   export common_password=<pick_one>
   export atc_password=<pick_one>
   ```

6. (Optional) Enable https support for concourse atc

  In `concourse.yml` under the atc properties block fill in the following fields:
  ```
  tls_bind_port: 443
  tls_cert: << SSL Cert for HTTPS >>
  tls_key: << SSL Private Key >>
  ```

7. Upload the cloud config:

  ```
  bosh update cloud-config cloud-config.yml
  ```

8. Target the deployment file and deploy:

  ```
  bosh deployment concourse.yml
  bosh deploy
  ```
# Let's connect to Concourse and try it out  
  
1. As per tradition, there is a simple [Hello, world!](https://concourse-ci.org/hello-world.html) tutorial for you to try. This will at least show the basics of [fly](https://concourse-ci.org/fly-cli.html) (the CLI for Concourse).

- Check whether you already have `fly` installed. If you don't, click [here](https://concourse-ci.org/fly-cli.html) to install `fly`.

```
$ fly -v
3.5.0
```

2. Connect to Concourse

```
$ fly -t concourse-rfm login -c http://35.197.133.183
logging in to team 'main'

WARNING: 
    fly version (3.5.0) is out of sync with the target (2.5.0). to sync up, run the following:
    fly -t concourse-rfm sync

username: concourse
password: password

target saved

$ fly -t concourse-rfm sync
downloading fly from http://35.197.133.183... 
 12.60 MB / 12.60 MB [===================================================================================================] 100.00% 11s
successfully updated from 3.5.0 to 2.5.0

$ fly -t concourse-rfm login -c http://35.197.133.183
username: concourse
password: password

target saved
```

3. Let's Execute _Hello World_

```
cd /work/concourse   # This is a directory I created
vi hello.yml
```

###### _hello.yml_
```
jobs:
- name: hello-world
  plan:
  - task: say-hello
    config:
      platform: linux
      image_resource:
        type: docker-image
        source: {repository: ubuntu}
      run:
        path: echo
        args: ["Hello, world!"]
```

4. Follow the steps shown below:

```
$ fly -t concourse-rfm set-pipeline -p hello-world -c hello.yml
jobs:
  job hello-world has been added:
    name: hello-world
    plan:
    - task: say-hello
      config:
        platform: linux
        image_resource:
          type: docker-image
          source:
            repository: ubuntu
        run:
          path: echo
          args:
          - Hello, world!
          dir: ""
    
apply configuration? [yN]: y
pipeline created!
you can view your pipeline here: http://35.197.133.183/teams/main/pipelines/hello-world

the pipeline is currently paused. to unpause, either:
  - run the unpause-pipeline command
  - click play next to the pipeline in the web ui
```

5. Let's proceed:

```
$ fly -t concourse-rfm unpause-pipeline -p hello-world
unpaused 'hello-world'

$ fly -t concourse-rfm unpause-job -j hello-world/hello-world
unpaused 'hello-world'
```

6. Final Step:
```
Goto: http://35.197.133.183/teams/main/pipelines/hello-world
```

# Congratulations: You have deployed Concourse on GCP

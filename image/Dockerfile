#
# thomasweise/docker-texlive-thin
#
# This is an image with a thin TeX Live installation and several
# fonts and tools.
# Source: http://github.com/thomasWeise/docker-texlive-thin/
# License: GNU GENERAL PUBLIC LICENSE, Version 3, 29 June 2007
# The license applies to the way the image is built, while the
# software components inside the image are under the respective
# licenses chosen by their respective copyright holders.
#
FROM ubuntu:20.04
MAINTAINER Thomas Weise <tweise@hfuu.edu.cn>

ENV LANG=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update &&\
# prevent doc and man pages from being installed
# the idea is based on https://askubuntu.com/questions/129566
    printf 'path-exclude /usr/share/doc/*\npath-include /usr/share/doc/*/copyright\npath-exclude /usr/share/man/*\npath-exclude /usr/share/groff/*\npath-exclude /usr/share/info/*\npath-exclude /usr/share/lintian/*\npath-exclude /usr/share/linda/*\npath-exclude=/usr/share/locale/*' > /etc/dpkg/dpkg.cfg.d/01_nodoc &&\
# remove doc files and man pages already installed
    rm -rf /usr/share/groff/* /usr/share/info/* &&\
    rm -rf /usr/share/lintian/* /usr/share/linda/* /var/cache/man/* &&\
    rm -rf /usr/share/man &&\
    mkdir -p /usr/share/man &&\
    find /usr/share/doc -depth -type f ! -name copyright -delete &&\
    find /usr/share/doc -type f -name "*.pdf" -delete &&\
    find /usr/share/doc -type f -name "*.gz" -delete &&\
    find /usr/share/doc -type f -name "*.tex" -delete &&\
    find /usr/share/doc -type d -empty -delete || true &&\
    mkdir -p /usr/share/doc &&\
    mkdir -p /usr/share/info &&\
# install utilities
    apt-get install -f -y --no-install-recommends apt-utils || exit 1 &&\
# get and update certificates, to hopefully resolve mscorefonts error
    apt-get install -f -y --no-install-recommends ca-certificates || exit 1  &&\
    update-ca-certificates &&\
# install some utilitites
    apt-get install -f -y --no-install-recommends \
          curl \
          fonts-dejavu \
          fonts-dejavu-core \
          fonts-dejavu-extra \
          fontconfig \
          xz-utils || exit 1  &&\
# install the microsoft core fonts
    echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections &&\
    echo "ttf-mscorefonts-installer msttcorefonts/present-mscorefonts-eula note" | debconf-set-selections &&\
    curl --output "/tmp/ttf-mscorefonts-installer.deb" "http://ftp.de.debian.org/debian/pool/contrib/m/msttcorefonts/ttf-mscorefonts-installer_3.7_all.deb" &&\
    apt install -f -y --no-install-recommends "/tmp/ttf-mscorefonts-installer.deb" || exit 1 &&\
    rm -f "/tmp/ttf-mscorefonts-installer.deb" &&\
# we make sure to contain the EULA in our container
    curl --output "/root/mscorefonts-eula" "http://corefonts.sourceforge.net/eula.htm" &&\
# install TeX Live and ghostscript as well as other tools
    apt-get install -f -y --no-install-recommends \
          cm-super \
          dvipng \
          ghostscript \
          make \
          latexmk \
          lmodern \
          poppler-utils \
          psutils \
          t1utils \
          tex-gyre \
          texlive-base \
          texlive-binaries \
          texlive-font-utils \
          texlive-fonts-recommended \
          texlive-latex-base \
          texlive-latex-recommended \
          texlive-luatex \
          texlive-pstricks \
          texlive-xetex || exit 1 &&\
# Install other necessary latex packages without
# user tree and then and clean up.
    tlmgr init-usertree || exit 1 &&\
    echo "tlmgr init-usertree done successfully" &&\
    updmap-sys || exit 1 &&\
    echo "initial updmap-sys done successfully" &&\
# tlmgr will call updmap-sys unsuccessfully for
# reasons I do not understand, so while it seemingly
# succeeds, it will still fail
    tlmgr install awesomebox \
                  babel-german \
                  background \
                  filehook \
                  fontawesome5 \
                  footmisc \
                  footnotebackref \
                  fvextra \
                  ly1 \
                  mdframed \
                  mweights \
                  needspace \
                  pagecolor \
                  siunitx \
                  sourcecodepro \
                  sourcesanspro \
                  ucharcat \
                  unicode-math \
                  xecjk || true &&\
    echo "tlmgr install completed" &&\
    cat /root/texmf/web2c/updmap.cfg >> /usr/share/texmf/web2c/updmap.cfg &&\
    rm -rf /root/texmf/tlpkg || true &&\
    rm -rf /root/texmf/web2c || true &&\
    cp -n -r /root/texmf/* /usr/share/texlive/texmf-dist/ || exit 1 &&\
    rm -rf /root/texmf || true &&\
    rm -rf /root/texmf-var || true &&\
    echo "installed packages merged and /root/texmf deleted" &&\
# update latex and font system files:
# Since I am not sure about the proper order,
# we just do this twice.
# iteration 1
    fc-cache -fv || exit 1 &&\
    echo "fc-cache succeeded" &&\
    texhash --verbose || exit 1 &&\
    echo "texhash completed successfully" &&\
    updmap-sys || exit 1 &&\
    echo "updmap-sys completed successfully" &&\
    mktexlsr --verbose || exit 1 &&\
    echo "mktexlsr succeeded" &&\
    fmtutil-sys --quiet --missing &&\
    echo "fmtutil-sys --missing completed successfully" &&\
    fmtutil-sys --quiet --all > /dev/null &&\
    echo "fmtutil-sys --all completed successfully" &&\
# iteration 2
    fc-cache -fv || exit 1 &&\
    echo "fc-cache succeeded" &&\
    texhash --verbose || exit 1 &&\
    echo "texhash completed successfully" &&\
    updmap-sys || exit 1 &&\
    echo "updmap-sys completed successfully" &&\
    mktexlsr --verbose || exit 1 &&\
    echo "mktexlsr succeeded" &&\
    fmtutil-sys --quiet --missing &&\
    echo "fmtutil-sys --missing completed successfully" &&\
    fmtutil-sys --quiet --all > /dev/null &&\
    echo "fmtutil-sys --all completed successfully" &&\
# Hopefully, the installation and merging of latex packages
# into the system-wide latex installation has worked...
# delete texlive sources and other potentially useless stuff
    rm -rf /usr/share/texmf/source || true &&\
    rm -rf /usr/share/texlive/texmf-dist/source || true &&\
    rm -rf /usr/share/texlive/texmf-dist/doc/ || true &&\
    find /usr/share/texlive -type f -name "readme*.*" -delete &&\
    find /usr/share/texlive -type f -name "README*.*" -delete &&\
    rm -rf /usr/share/texlive/release-texlive.txt || true &&\
    rm -rf /usr/share/texlive/doc.html || true &&\
    rm -rf /usr/share/texlive/index.html || true &&\
    rm -rf /usr/share/texlive/texmf-dist/fonts/source &&\
    rm -rf /usr/share/texlive/texmf-dist/tex/latex/pst-poker &&\
# clean up all temporary files
    apt-get clean -y &&\
    rm -rf /var/lib/apt/lists/* &&\
    rm -f /etc/ssh/ssh_host_* &&\
# delete man pages and documentation
    rm -rf /usr/share/man &&\
    mkdir -p /usr/share/man &&\
    find /usr/share/doc -depth -type f ! -name copyright -delete &&\
    find /usr/share/doc -type f -name "*.pdf" -delete &&\
    find /usr/share/doc -type f -name "*.gz" -delete &&\
    find /usr/share/doc -type f -name "*.tex" -delete &&\
    find /usr/share/doc -type d -empty -delete || true &&\
    mkdir -p /usr/share/doc &&\
    rm -rf /var/cache/apt/archives &&\
    mkdir -p /var/cache/apt/archives &&\
    rm -rf /tmp/* /var/tmp/* &&\
    find /usr/share/ -type f -empty -delete || true &&\
    find /usr/share/ -type d -empty -delete || true &&\
    mkdir -p /usr/share/texmf/source &&\
    mkdir -p /usr/share/texlive/texmf-dist/source

ADD scripts /bin/

# ROOT version in the MAJOR.MINOR.REVISION format or a https://github.com/root-project/root branch name / tag. Example: ROOT_VERSION=6.26.00
ARG ROOT_VERSION=latest-stable
# C++ standard to compile ROOT. Example: CMAKE_CXX_STANDARD=17
ARG CMAKE_CXX_STANDARD=17

FROM ubuntu:22.04

LABEL maintainer.name="Yongjie Deng"
LABEL maintainer.email="deng_yongjie@foxmail.com"

ENV LANG=C.UTF-8



COPY packages packages

RUN apt-get update -qq && \
    ln -sf /usr/share/zoneinfo/UTC /etc/localtime && \
    apt-get -y install $(cat packages) wget git && \
    apt-get autoremove -y && \
    apt-get clean -y && \
    rm -rf /var/cache/apt/archives/* && \
    rm -rf /var/lib/apt/lists/*

RUN yes | unminimize


ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
ENV PYTHONPATH=/usr/local/lib/python3.10/site-packages:$PYTHONPATH

RUN ROOT_GIT_URL=https://github.com/root-project/root.git \
    && if [ -z "$(git ls-remote --heads $ROOT_GIT_URL $ROOT_VERSION)" ] ; then \
    export ROOT_GIT_REVISION="v$(echo $ROOT_VERSION | cut -d. -f1)-$(echo $ROOT_VERSION | cut -d. -f2)-$(echo $ROOT_VERSION | cut -d. -f3)" \
    ; else \
    export ROOT_GIT_REVISION=$ROOT_VERSION \
    ; fi \
    # Above lines will set ROOT_GIT_REVISION to ROOT_VERSION argument if it corresponds to a valid branch name (such as ROOT_VERSION=master),
    # otherwise it will assume its a semantic version string and try to convert it into the tag format (such as 6.26.00 -> v6-26-00)
    && git clone --branch $ROOT_GIT_REVISION --depth=1 $ROOT_GIT_URL /tmp/source \
    && cd /tmp/source \
    && mkdir -p /tmp/build &&  cd /tmp/build \
    && cmake /tmp/source \
    -DCMAKE_CXX_STANDARD=$CMAKE_CXX_STANDARD \
    -Dgnuinstall=ON \
    -Dsoversion=ON \
    # For ROOT version 6.26.00 it is necessary to set `-Druntime_cxxmodules=OFF` (https://github.com/root-project/root/pull/10198)
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_INSTALL_BINDIR=bin \
    -DCMAKE_INSTALL_CMAKEDIR=lib/x86_64-linux-gnu/cmake/ROOT \
    -DCMAKE_INSTALL_DATAROOTDIR=share \
    -DCMAKE_INSTALL_DATADIR=share/root \
    -DCMAKE_INSTALL_DOCDIR=share/doc/root \
    -DCMAKE_INSTALL_ELISPDIR=share/emacs/site-lisp \
    -DCMAKE_INSTALL_FONTDIR=share/root/fonts \
    -DCMAKE_INSTALL_ICONDIR=share/root/icons \
    -DCMAKE_INSTALL_INCLUDEDIR=include/ROOT \
    -DCMAKE_INSTALL_JSROOTDIR=share/root/js \
    -DCMAKE_INSTALL_LIBDIR=lib/x86_64-linux-gnu \
    -DCMAKE_INSTALL_MACRODIR=share/root/macros \
    -DCMAKE_INSTALL_MANDIR=share/man \
    -DCMAKE_INSTALL_OPENUI5DIR=share/root/ui5 \
    -DCMAKE_INSTALL_PYTHONDIR=lib/python3/dist-packages \
    -DCMAKE_INSTALL_SRCDIR=/dev/null \
    -DCMAKE_INSTALL_SYSCONFDIR=/etc/root \
    -DCMAKE_INSTALL_TUTDIR=share/root/tutorials \
    && make install \
    && rm -rf /tmp/build /tmp/source


RUN url='https://gitlab.cern.ch/hepmc/HepMC3/-/archive/3.2.7/HepMC3-3.2.7.tar.gz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz && \
    cd 'HepMC3-3.2.7' && \
    cmake . -DHEPMC3_ENABLE_ROOTIO=NO  && \
    make && \
    make install && \
    cd - && \
    rm -rf 'HepMC3-3.2.7'

RUN url='https://fastjet.fr/repo/fastjet-3.4.3.tar.gz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz && \
    cd 'fastjet-3.4.3' && \
    ./configure --enable-pyext --enable-allcxxplugins && \
    make && \
    make install && \
    cd - && \
    rm -rf 'fastjet-3.4.3'

RUN url='http://fastjet.hepforge.org/contrib/downloads/fjcontrib-1.055.tar.gz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz && \
    cd 'fjcontrib-1.055' && \
    ./configure && \
    make && \
    make install && \
    make fragile-shared-install && \
    cd - && \ 
    rm -rf 'fjcontrib-1.055'

RUN url='https://yoda.hepforge.org/downloads?f=YODA-1.9.9.tar.gz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz && \
    cd 'YODA-1.9.9' && \
    PYTHON_VERSION=3 ./configure --disable-root && \
    make && \
    make install && \
    cd - && \
    rm -rf 'YODA-1.9.9'

RUN url='https://lhapdf.hepforge.org/downloads/?f=LHAPDF-6.5.4.tar.gz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz && \
    cd 'LHAPDF-6.5.4' && \
    PYTHON_VERSION=3 ./configure && \
    make && \
    make install && \
    /usr/bin/mkdir -p /usr/local/lib/python3.11/dist-packages/ && \
    cp -r /root/LHAPDF-6.5.4/wrappers/python/build/lhapdf /usr/local/lib/python3.11/dist-packages/ && \
    cd - && \
    rm -rf 'LHAPDF-6.5.4'

RUN url='https://rivet.hepforge.org/downloads/?f=Rivet-3.1.8.tar.gz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz && \
    cd 'Rivet-3.1.8' && \
    ./configure --with-hepmc3=`HepMC3-config --prefix` && \
    make && \
    make install && \
    cd - && \
    rm -rf 'Rivet-3.1.8'

RUN url='https://launchpad.net/mg5amcnlo/3.0/3.6.x/+download/MG5_beta_v3.6.0.tgz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz 

RUN url='https://pythia.org/download/pythia83/pythia8312.tgz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz && \
    cd 'pythia8312' && \
    ./configure --prefix=/usr/local --with-python --with-python-include=/usr/include/python3.10 --with-hepmc3 --with-lhapdf6 --with-rivet --with-gzip && \
    make && \
    make install && \
    cd - && \
    rm -rf 'pythia8312'

RUN ln -sf /usr/bin/python3 /usr/bin/python

RUN sed -i.bak -e '97d' /etc/ImageMagick-6/policy.xml

RUN lhapdf install NNPDF23_lo_as_0130_qed


RUN url='https://www.nikhef.nl/~h24/qcdnum-files/download/qcdnum180000.tar.gz' && \
    cd /root && \
    curl -sL "$url" | \
    tar xz && \
    cd 'qcdnum-18-00-00' && \
    ./configure --prefix=/usr/local && \
    make && \
    make install && \
    cd - && \
    rm -rf 'qcdnum-18-00-00'



RUN cd /root && \
    echo n | svn checkout --username anonymous --password anonymous svn://powhegbox.mib.infn.it/trunk/POWHEG-BOX-V2 && \
    cd POWHEG-BOX-V2 && \
    echo n | svn checkout --username anonymous --password anonymous svn://powhegbox.mib.infn.it/trunk/User-Processes-V2/hvq && \
    cd hvq && \
    make && \
    mv pwhg_main pwhg_main-hvq 

RUN cd /root/POWHEG-BOX-V2 && \
    echo n | svn checkout --username anonymous --password anonymous svn://powhegbox.mib.infn.it/trunk/User-Processes-V2/Z && \
    cd Z && \
    sed -i.bak -e "s/ANALYSIS=default/ANALYSIS=none/" -e "s/^PWHGANAL=pwhg_analysis-dummy.o$/PWHGANAL=pwhg_analysis-dummy.o pwhg_bookhist.o/" Makefile && \
    make && \
    mv pwhg_main pwhg_main-Z


ENV PATH=/root/POWHEG-BOX-V2/hvq:/root/POWHEG-BOX-V2/Z:$PATH
ENV PATH=$PATH:/usr/local/bin:/root/MG5_aMC_v3_5_5/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/
ENV PYTHONPATH=$PYTHONPATH:/usr/local/lib/python3.11/site-packages:/usr/local/lib:/usr/lib/python3/dist-packages/:/usr/local/local/lib/python3.11/dist-packages

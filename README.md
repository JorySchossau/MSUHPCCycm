## Installing [YouCompleteMe](https://github.com/Valloric/YouCompleteMe) on MSU's HPCC
----------
*Note: if you are using another computing cluster then the details may differ.*

Begin by load the following modules and ensuring they'll always load on login.

    echo module load GNU/4.8.2 CMake/3.1.0 Python/2.7.2 Clang >> ~/.bashrc
    source ~/.bashrc # force reload with our new modules
Grab a suitable version of Vim and prepare a symbolic link because it will look for `python2` in your `PATH` during configuration for building.

    wget ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2
    tar xfj vim-7.4.tar.bz2
    cd vim74
    ln -s /opt/software/Python/2.7.2--GCC-4.4.5/bin/python2.7 /mnt/home/${USER}/bin/python2
Now build Vim and install to your `~/bin` dir.

    ./configure --with-features=huge --enable-multibyte --enable-pythoninterp --with-python-config-dir=/opt/software/Python/2.7.2--GCC-4.4.5/lib/python2.7/config --enable-cscope --enable-cscope --prefix=/mnt/home/${USER}
    make -j4
    make install
If you mess up the configure and it detects the wrong files then those findings will be cached in vim74/src/auto/config.cache and you can just delete that to force a refresh on next configure.

Put the following at the top of your `~/.vimrc`.

    syntax on
    set clipboard=unnamed
    set number
    set tabstop=4 shiftwidth=4 expandtab
    
    set nocompatible
    filetype off
    
    set rtp+=~/.vim/bundle/vundle/
    call vundle#rc()
    
    Bundle 'gmarik/vundle'
    Bundle 'Valloric/YouCompleteMe'
    
    filetype plugin indent on
Grab the source code for YCM, perform a manual build, and initialize and install all remaining plugins.

    git clone https://github.com/Valloric/YouCompleteMe.git ~/.vim/bundle/YouCompleteMe
    cd ~/.vim/bundle/YouCompleteMe
    git submodule update --init --recursive
    
    cd ~/.vim/bundle/YouCompleteMe
    mkdir ycm_build
    cd ycm_build
    cmake -G "Unix Makefiles" . ~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp -DPYTHON_INCLUDE_DIR=/opt/software/Python/2.7.2--GCC-4.4.5/include -DPYTHON_LIBRARY=/opt/software/Python/2.7.2--GCC-4.4.5/lib/libpython2.7.so -DUSE_CLANG_COMPLETER=1 -DPYTHON_EXECUTABLE=/opt/software/Python/2.7.2--GCC-4.4.5/bin/python -DEXTERNAL_LIBCLANG_PATH=/opt/software/Clang/20140106--GCC-4.8.2/lib/libclang.so
    
    make ycm_support_libs -j4
    vim +PluginInstall +qall
Don't forget to add a copy of [`.ycm_extra_conf.py`](https://raw.githubusercontent.com/Valloric/ycmd/master/cpp/ycm/.ycm_extra_conf.py) to each project's source dir, and tweak it.
1. 标准IO和文件IO都是用来进行输入输出操作的，但它们的实现方式不同，主要区别如下：

  1. 缓冲方式不同：标准IO是带缓冲的，而文件IO是不带缓冲的。标准IO在进行输入输出操作时，会将数据先存储到缓冲区中，等到缓冲区满了或者遇到换行符等特殊情况时才进行实际的输入输出操作。而文件IO则是直接进行输入输出操作。
  2. 实现方式不同：标准IO是通过标准IO库函数实现的，而文件IO是通过系统调用实现的。标准IO库函数提供了一组易用的API，包括printf、scanf等，而文件IO则需要使用系统调用函数，如read、write、open等。
  3. 可移植性不同：标准IO是跨平台的，而文件IO则受操作系统的限制。标准IO库函数在不同的操作系统中表现一致，而文件IO则需要根据不同的操作系统进行适配。
  4. 错误处理方式不同：标准IO和文件IO在遇到错误时的处理方式也不同。标准IO会返回特殊值来表示错误一般会置位错误码，而文件IO则会设置errno变量(置位错误码)来表示错误，并返回-1。

  总的来说，标准IO相对于文件IO来说更加方便和易用，但是在一些特殊情况下，如需要进行低级别的文件操作时，文件IO可能更加适合。

  如果需要进行文件访问和处理，且不需要高精度输出和跨平台兼容性，可以选择使用文件IO；如果需要进行高精度输出和跨平台兼容性，或者需要快速开发和简单应用，可以选择使用标准IO。如果需要在多线程环境下使用IO，需要谨慎选择并使用线程同步机制来保证安全。

2. inode号是什么？(linux系统中一切皆文件！)
  inode（index node）是Linux文件系统中的一个概念，用于描述文件的元数据信息，包括文件的访问权限、所有者、大小、修改时间、创建时间等。

  每个文件都会对应一个唯一的inode节点，inode节点包含了文件的元数据信息，同时还包含了文件数据在硬盘上的物理位置等信息。因此，通过inode节点可以访问和管理文件的元数据信息和数据内容。

  在Linux系统中，可以使用ls -i命令来查看文件的inode号，inode号是一个整数值，用于唯一标识文件。当使用文件名访问文件时，系统会通过文件名查找对应的inode节点，然后根据inode节点中的信息来访问和管理文件。（命令：ls -i）

  需要注意的是，inode节点是文件系统的概念，并不是所有的文件系统都使用inode节点来管理文件。例如，FAT32文件系统就不使用inode节点来管理文件，而是使用文件的簇号来访问和管理文件。

3. 
  /dev/null?
  在Linux系统中，/dev/null是一个特殊的设备文件，也称为空设备。它是一个黑洞设备，所有写入该设备的数据都会被直接丢弃，而所有从该设备读取的数据都为EOF（结束符）。

  /dev/null常常被用于将不需要的输出或错误信息重定向到该设备中，从而实现快速丢弃这些信息。例如，如果运行一个命令时不想要它的输出，可以使用重定向符将输出重定向到/dev/null中，例如：

  ```shell
  $ command > /dev/null
  ```

  这样，命令的输出将被丢弃，不会在终端上显示。同样地，如果想要将错误信息重定向到/dev/null中，可以使用2>符号，例如：

  ```shell
  $ command 2> /dev/null
  ```

  这样，命令的错误信息将被丢弃，不会在终端上显示。


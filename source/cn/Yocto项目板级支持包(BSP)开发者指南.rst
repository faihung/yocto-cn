====================================================================
Yocto 项目软件开发套件（SDK）开发者指南
====================================================================

.. note::

   翻译进行中...


* 链接： `Yocto Project Board Support Package (BSP) Developer's Guide <http://www.yoctoproject.org/docs/1.6.1/bsp-guide/bsp-guide.html>`_
* 版本： 1.6.1


板载支持包（BSP）开发者指南
====================================================================

1.1. BSP 层（Layer）
--------------------------------------------------------------------

BSP 在基础目录中包含了一个文件结构。当然，你可以将基础目录和文件结构认为是 BSP 层。在 Yocto 项目中，尽管没有非常严格的规定，但是层的命名遵循以下规范：

.. code::

    meta-<bsp_name>


字符串 “meta-” 通常预指为设备或者平台名称，格式为上述的 “bsp_name”。

BSP 层的基础目录（`meta-<bsp_name>`）是根目录，也就是在构建目录中 *conf/bblayers.conf* 文件中 *BBLAYERS* 变量中添加的路径位置。在添加该层根路径后，将允许 OpenEmbedded 构建系统识别 BSP 定义，并通过它来构建镜像。示例如下：

.. code::

     BBLAYERS ?= " \
       /usr/local/src/yocto/meta \
       /usr/local/src/yocto/meta-yocto \
       /usr/local/src/yocto/meta-yocto-bsp \
       /usr/local/src/yocto/meta-mylayer \
       "

     BBLAYERS_NON_REMOVABLE ?= " \
       /usr/local/src/yocto/meta \
       /usr/local/src/yocto/meta-yocto \
       "

某些 BSP 在 BSP 的根层之上可能还需要其他的层。在这些情况下，您需要将这些层也添加到 **BBLAYERS** 以实现 BSP 的构建。您也必须在 BSP 的 README 文件中的 “Dependencies” 部分对需要的其他层进行说明，以及在 README 文件中可能包含的构建指令部分加以说明。

还有一些层会作为其他 BSP 层的汇总。例如 meta-intel 层，该层包含了多个独立的 BSP 层。

更多有关 BSP 层的详细信息，可以查看 Yocto 项目开发手册的 “理解和创建层” 部分。

1.2. 文件系统布局示例
--------------------------------------------------------------------

1.2.1. 授权文件
********************************************************************

在 BSP 层中，您可能会发现如下文件：

.. code::

    meta-<bsp_name>/<bsp_license_file>


该文件指明了该 BSP 的授权要求。这里的文件类型可能会根据授权要求的不同而有所变化。例如，对于 Crown Bay 的 BSP 包，所有的授权要求都在 COPYING.MIT 文件。

授权文件可能是 MIT，BSD，GPLv* 等等。我们建议对 BSP 包添加授权文件，但是并不强求，授权以及使用何种授权完全取决于 BSP 的开发人员。


1.2.2. README 文件
********************************************************************

在 BSP 包中还包含如下文件：

.. code::

    meta-<bsp_name>/README


该文件可能提供了有关 `binary/` 目录中包含的启动镜像如何引导等信息，也可能会提供有关镜像构建所需要的特殊信息。

最基本的，README 文件会包含一个依赖关系列表，例如该 BSP 包依赖的其他层的名称，以及该 BSP 包的维护人员的联系信息。

1.2.3. README.sources 文件
********************************************************************

在 BSP 包中还会包含如下文件：

.. code::

    meta-<bsp_name>/README.sources


该文件提供了如何定位 BSP 源码文件等信息。例如，提供了哪里可以找到该 BSP 包中包含的镜像的源码等信息。当然这些信息还会包括帮助您找到生成镜像相关的 **元数据** 信息。

1.2.4. 预编译用户二进制文件
********************************************************************

1.2.5. 层（Layer）配置文件
********************************************************************

在 BSP 包中还会包含如下文件：

.. code::

    meta-<bsp_name>/conf/layer.conf


`conf/layer.conf` 文件指明了该文件结构为一个层，该层的内容，以及构建系统中如何使用等信息。通常，标准的模板文件如下所示。在下面的示例中，您应该讲 “bsp” 以及 “_bsp” 替换为相应的 BSP 包的实际名称（例如，示例末班中的 `<bsp_name>`）。


.. code::

     # We have a conf and classes directory, add to BBPATH
     BBPATH .= ":${LAYERDIR}"

     # We have a recipes directory, add to BBFILES
     BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
             ${LAYERDIR}/recipes-*/*/*.bbappend"

     BBFILE_COLLECTIONS += "bsp"
     BBFILE_PATTERN_bsp = "^${LAYERDIR}/"
     BBFILE_PRIORITY_bsp = "6"


为了更好的阐明字符串的替换机制，下面是 Crown Bay 包的 `conf/layer.conf` 文件：

.. code::

     BBFILE_COLLECTIONS += "crownbay"
     BBFILE_PATTERN_crownbay = "^${LAYERDIR}/"
     BBFILE_PRIORITY_crownbay = "6"


该文件让 BitBake 容易识别到相关的 recipe 和配置目录。该文件必须存在，以便于 OpenEmbedded 构建系统对 BSP 包的识别。

1.2.6. 硬件配置选项
********************************************************************

在 BSP 包中还存在一下文件：

.. code::

    meta-<bsp_name>/conf/machine/*.conf


机器设备文件讲 BSP 中其他位置包含的所有信息以一种格式绑定在一起，让构建系统更为方便的理解。如果 BSP 包支持多中设备，那么这里需要显示多个设备配置文件。这些文件名称与用户设置的 `MACHINE` 变量相对应。

这些文件中定义的内容包括使用的内核报（虚拟内核的 `PREFERRED_PROVIDER`），不同类型镜像中包含的硬件驱动，其他所需要的所有特殊软件组件，任何有关引导程序的信息，以及所有有关镜像格式的特殊要求。

每个 BSP 包至少需要一个设备文件。不过，您可以提供多个文件。

crownbay.conf 文件还包含了硬件的 "调配" 文件，该文件通常用于定义包的结构，以及致命优化标记，对于标记需要根据制定的处理器进行选择以便提供最优的性能。

“调配” 文件可以在 **源目录** 的 `meta/conf/machine/include` 中找到。例如对于 ia32-base.inc 文件，就位于 ` meta/conf/machine/include` 目录。

如果希望使用包含文件，您只需要在设备配置文件中包含相关文件即可。例如，Crown Bay 的 BSP 包，它的 `crownbay.conf` 文件就包含了下面的文件：

.. code::

     require conf/machine/include/intel-core2-32-common.inc
     require conf/machine/include/meta-intel.inc
     require conf/machine/include/meta-intel-emgd.inc


1.2.7. 各种 BSP 特别的 Recipe 文件
********************************************************************

在 BSP 包中也存在如下文件：

.. code::

     meta-<bsp_name>/recipes-bsp/*


其他的目录包含了一些 BSP 包所具有的各种各样的 recipe 文件。最需要注意的是 formfactor 文件。例如，在 Crown Bay 的 BSP 包中，就存在 `formfactor_0.0.bbappend` 文件，是一个附加文件，用于声明启动构建的 recipe 。此外，还有一些构建过程中设备所特有的设置文件，由 `machconfig` 文件所定义。例如，对于 Crown Bay，就存在两个 `machconfig` 文件：一个支持 the Intel® Embedded Media and Graphics Driver (Intel® EMGD) ，而另一个则不然：

.. code::

     meta-crownbay/recipes-bsp/formfactor/formfactor/crownbay/machconfig
     meta-crownbay/recipes-bsp/formfactor/formfactor/crownbay-noemgd/machconfig
     meta-crownbay/recipes-bsp/formfactor/formfactor_0.0.bbappend
 

.. note::

    如果一个 BSP 包不存在 formfactor 入口，默认将会根据安装在主 formfactor 中的配置文件启动，即 **源目录** 中的 `meta/recipes-bsp/formfactor/formfactor_0.0.bb` 。


1.2.8. 显示支持文件
********************************************************************

在 BSP 包中还存在如下支持文件：

.. code::

     meta-<bsp_name>/recipes-graphics/*


如果某个 BSP 包对图形支持有特殊的需要，该目录将会包含相关的 recipe 文件。BSP 用以支持显示的所需要的所有文件都保存在这里。例如，对于 Crown Bay ， `xorg.conf` 文件用以检测所需要的图形支持（如，Intel® Embedded Media 图形驱动 (EMGD) 或者 Video Electronics Standards Association (VESA) 图形显示）：

.. code::

     meta-crownbay/recipes-graphics/xorg-xserver/xserver-xf86-config_0.1.bbappend
     meta-crownbay/recipes-graphics/xorg-xserver/xserver-xf86-config/crownbay/xorg.conf


1.2.9. Linux 内核配置
********************************************************************

1.3. BSP 发布要求和建议
--------------------------------------------------------------------

1.3.1. BSP 发布要求
********************************************************************

1.3.2. BSP 发布建议
********************************************************************

如下是符合 Yocto 项目约定的，对发布的 BSP 的建议：

* **可启动镜像：**
* **使用 Yocto Linux 内核：** 在 BSP 包中使用的内核 recipe 文件应该基于 Yocto 项目的 Linux 内核。基于这些内核构建您的 recipe 文件，将会减轻管理 BSP 的负担，同时增加弹性。有关这些内核的信息，请查看 **[源码库](http://git.yoctoproject.org/cgit.cgi)** 中 **Yocto Linux Kernel** 部分。 

1.4. BSP 的 Recipe 定制
--------------------------------------------------------------------

如果您打算定制一个独立的 BSP 包，需要执行下述操作：

* 为修改的 recipe 创建 .bbappend 文件。有关如何使用添加文件的信息，请参考 Yocto 项目开发手册中的 " .bbappend 文件使用方法" 部分。

* 确保支持您的硬件设备的 BSP 包中的目录结构遵循如下的结构以便构建系统能够找出。具体示例请参考后续部分的介绍。
* 将目录中添加文件的名称和设备名称相匹配，并且位置位于 BSP 包中相应的子目录（例如，`recipe-bsp`，`recipe-graphics`，`recipe-core`，等等）
* 讲命名为您的设备名称的 BSP 特殊文件放在 BSP 包。

如下示例将会帮助您更好的理解该过程。我们以一个示例作为参考，需要通过添加命名为 `interfaces` 的 BSP 特殊配置文件到 “xyz” 设备的 `init-ifupdown_1.0.bb` 文件来定制 recipe 。那么操作过程如下：

1. 编辑 `init-ifupdown_1.0.bbappend` 文件，确保包含下面内容：

   .. code::

       FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
     
    添加文件应该位于 `meta-xyz/recipes-core/init-ifupdown` 目录。
2. 在 BSP 包的下述位置新建 `interfaces` 配置文件：

   .. code::

      meta-xyz/recipes-core/init-ifupdown/files/xyz/interfaces
     
    在添加配置文件中的 `FILESEXTPAPATHS` 变量将会扩展构建系统在构建过程中用以搜索文件的搜索路径。因此，在本示例中，您需要在添加文件所在位置创建 `files` 目录。

1.5. BSP 授权思考
--------------------------------------------------------------------

1.6. 使用 Yocto 项目 BSP 工具
--------------------------------------------------------------------

Yocto 项目包含了一系列的工具能够帮助您从无到有简历 [BSP 包](http://www.yoctoproject.org/docs/1.6.1/bsp-guide/bsp-guide.html#bsp-layers)，无需查看 Metadata 文件即可完成内核的基本配置和管理。这些工具包括 yocto-bsp 和 yocto-kernel 。

下面部分，将会详细介绍有关 yocto-bsp 和 yocto-kernel 工具的位置和所提供的功能。

1.6.1. 通用功能
********************************************************************

1.6.2. 使用 yocto-bsp 脚本创建新的 BSP 层
********************************************************************

yocto-bsp 脚本用以创建 Yocto 项目支持的任何架构的 BSP 包，包括 QEMU 版本。该脚本的默认模式是给您提供一些用于生成 BSP 包所需要的信息。

对于目前的 BSP 包，该脚本将会为您提供下面的不同重要的参数：

* 使用的内核
* 使用的内核补丁（重用）
* 是否使用 X， 以及所用的驱动
* 是否开启 SMP
* 是否支持键盘
* 是否支持触摸屏
* 是否保留 BSP 相关的配置项

1.6.3. 使用 yocto-kernel 创建内核补丁和配置项
********************************************************************

我们假定您已经使用 yocto-bsp 创建了一个 BSP 包，并且将其添加到了 `bblayers.conf` 文件中的 `BBLAYERS` 变量。那么现在您可以使用 yocto-kernel 脚本来添加有关 BSP 内核的补丁和配置项。

yocto-kernel 脚本允许您添加，删除，和列出补丁及BSP 内核的内核配置 .bbappend 文件。所有您需要做的就是使用相应的子命令。需要说明的是，最简单的查看可供是用的子命令是使用 yocto-kernel 的内置帮助信息：

.. code:: shell

     $ yocto-kernel
     Usage:

      Modify and list Yocto BSP kernel config items and patches.

      usage: yocto-kernel [--version] [--help] COMMAND [ARGS]

      Current 'yocto-kernel' commands are:
        config list       List the modifiable set of bare kernel config options for a BSP
        config add        Add or modify bare kernel config options for a BSP
        config rm         Remove bare kernel config options from a BSP
        patch list        List the patches associated with a BSP
        patch add         Patch the Yocto kernel for a BSP
        patch rm          Remove patches from a BSP
        feature list      List the features used by a BSP
        feature add       Have a BSP use a feature
        feature rm        Have a BSP stop using a feature
        features list     List the features available to BSPs
        feature describe  Describe a particular feature
        feature create    Create a new BSP-local feature
        feature destroy   Remove a BSP-local feature

      See 'yocto-kernel help COMMAND' for more information on a specific command.



     Options:
       --version    show program's version number and exit
       -h, --help   show this help message and exit
       -D, --debug  output debug information


`yocto-kernel patch add` 子命令允许您为 BSP 包添加一个补丁。下面的示例用于给 `myarm` BSP 包添加两个补丁：


.. code:: shell

     $ yocto-kernel patch add myarm ~/test.patch
     Added patches:
             test.patch

     $ yocto-kernel patch add myarm ~/yocto-testmod.patch
     Added patches:
             yocto-testmod.patch


.. note::

    尽管前一个示例中，我们一次添加一个补丁，其实我们完全可以一次添加多个补丁。


您可以使用 `yocto-kernel patch list` 子命令查看已经添加的补丁文件。示例如下：

.. code:: shell

     $ yocto-kernel patch list myarm
     The current set of machine-specific patches for myarm is:
             1) test.patch
             2) yocto-testmod.patch


我们也可以使用 `yocto-kernel patch rm` 子命令移除指定补丁。示例如下：

.. code:: shell

     $ yocto-kernel patch rm myarm
     Specify the patches to remove:
             1) test.patch
             2) yocto-testmod.patch
     1
     Removed patches:
             test.patch


接下来，我们使用 `yocto-kernel patch list` 子命令，再次查看相关补丁是否已经确切的删除。

.. code:: shell

     $ yocto-kernel patch list myarm
     The current set of machine-specific patches for myarm is:
             1) yocto-testmod.patch


以完全相似的方式，您可以使用 `yocto-kernel config add` 子命令添加一个或者多个内核配置项设置。下述的命令讲添加一系列的配置项到 myarm BSP包：

.. code:: shell

     $ yocto-kernel config add myarm CONFIG_MISC_DEVICES=y
     Added items:
             CONFIG_MISC_DEVICES=y

     $ yocto-kernel config add myarm CONFIG_YOCTO_TESTMOD=y
     Added items:
             CONFIG_YOCTO_TESTMOD=y                    


.. note::

    尽管上个示例中，我们每次只添加一个配置项，其实我们可以一次添加多个配置项。


您可以列出 BSP 包相关的配置项。执行下述命令将会显示您或其他人为 BSP 包所添加的配置项：

.. code:: shell

     $ yocto-kernel config list myarm
     The current set of machine-specific kernel config items for myarm is:
             1) CONFIG_MISC_DEVICES=y
             2) CONFIG_YOCTO_TESTMOD=y


最后，您也可以以一种完全相同于 `yocto-kernel patch rm` 的方式，使用 `yocto-kernel config rm` 子命令删除一个或多个配置项。
WIP - CMake Primer
------------------
The goal of this primer is to give an overview of the fundamental concepts of CMake.  CMake is a complex tool and it is easy to get lost in the noise.  By being grounded in these fundamentals, your learning process should be much easier.

## Motivation for CMake
Ugh, what's this?  Yet another build system?  Is this just another _hot new thing_ that everyone is going to forget about in 5 years?  Why should _I_ learn this?  What's in it _for me_?

To understand CMake, we need to understand the problems it's trying to solve.  As pointed out by Hayt in [this Stack Overflow thread](https://stackoverflow.com/questions/40083642/why-do-we-need-cmake), "building things in C++ on different platforms is a mess currently."  That is one of CMake's main selling points, that it's cross-platform.  CMake was started by Bill Hoffman [in response to the need for a cross-platform build environment](https://cmake.org/overview/).

Another issues is that Makefiles on their own [kinda sorta suck](https://dmitryfrank.com/blog/2016/1211_i_am_tired_of_makefiles).  To really be useful they need to be used together with the Unix configure tool or GNU's autoconf, but that's not very portable.  This is another selling point for CMake; it pulls in the configuration functionality.  Running CMake automatically performs the configure stage.

Finally, the world of embedded development is quickly reaching an impasse.  Forget yet another build system.  What I really don't want is _yet another IDE_.  I don't understand why it is that every vendor feels the need to spin their own version of Eclipse when at the end of the day its all just variations of the same wrapper around GCC.  I think there's still this expectation in embedded development that your editor _is_ your build tool.  [Wrong wrong wrong!](https://blog.codinghorror.com/the-f5-key-is-not-a-build-process/)  These are separate concepts.  The build system should be standardizable, codifiable, executable from the command line, and repeatable across machines and from CI (cloud-based or local).  As far as the editor, you can use Notepad for all I care, as long as it doesn't affect the build.  And yes, you _can_ use VS Code and Eclipse as standalone editors and debuggers, _**as long as it doesn't affect the build**_.

Besides the umpteen skinned Eclipses, there is an even bigger problem with the proprietary IDEs.  Yes, I'm talking about the big dogs in the embedded industry, IAR and Keil.  [ARM](https://www.arm.com/) is still pushing [a proprietary IDE](https://developer.arm.com/tools-and-software/embedded/arm-development-studio) on their developer website, which from what I can tell is just the next iteration of Keil.  Don't be fooled.  The GNU ARM toolchain is [available for free](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads).

The main problem with these proprietary IDEs is how you handle licensing in CI.  These IDEs are premised on the idea of node-locked, dongle-locked, or FlexLM based licenses.  Even Segger Studio, which is "free for Nordic processors" still encounters a problem with CI.  To get the free license, you are required to go through a [node-based registration process](https://wiki.segger.com/Get_a_License_for_Nordic_Semiconductor_Devices).  _Give with one hand.  Take with the other._

So, where am I going with this monologue?

Right... we need a sophisticated, modern command-line based build system which can handle configuration properly and for which we can develop templates that can be reused across projects.  Once we have this, we can forget about the endless deluge of custom IDEs or the intransigent licensing problems that threaten to derail our modernization attempts.  Then at least we stand a chance of developing [mature processes](https://www.electronvector.com/blog/a-maturity-test-for-firmware-organizations).

## CMake - The 10,000 Foot View
LLVM's [CMake Primer](https://llvm.org/docs/CMakePrimer.html#ft-view) describes CMake as
> a tool that reads script files in its own language that describe how a software project builds. As CMake evaluates the scripts it constructs an internal representation of the software project. Once the scripts have been fully processed, if there are no errors, CMake will generate build files to actually build the project. CMake supports generating build files for a variety of command line build tools as well as for popular IDEs.

Let's unpack that a bit.  CMake is not the build system itself.  It generates build files for the build system of your choice: make, ninja, MSVC, Xcode, whatever.  Furthermore, CMake has _configuration_ and _generation_ stages.  The configuration stage is similar to autoconf.  During this stage the CMake scripts are processed and checks performed resulting ultimately in a set of information that can be passed to a back-end generator.  A generator exists for each build system that CMake supports.  The generator must be specified when CMake is invoked, otherwise a platform-specific default will be chosen.

![CMake Stages](resources/CMake-Stages.drawio.png)

CMake is designed only to be rerun as needed.  Once the build files are generated, it is necessary only to invoke the targeted build system (e.g. make or ninja).  The generated build files include dependencies that will rerun CMake _only as needed_.  That is, if you modify the CMakeLists.txt *.cmake files, the CMake command will be rerun, otherwise it will not be.  Furthermore, information about the configuration is cached in the CMakeCache.txt file in your build output folder.  All of this serves to make CMake fast and fully supportive of incremental builds.


-----
Copyright &copy; 2022 Aaron Fontaine

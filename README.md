# PreEmptive Protection for iOS - Sample App: Vie

This project is provided as an example of how to integrate the software obfuscation tool [*PreEmptive Protection for iOS*](https://www.preemptive.com/products/ppios) from PreEmptive Solutions into an existing iOS app's Xcode project.

This project was forked from [Vie](https://github.com/fabienwarniez/Vie) by Fabien Warniez who graciously made the project available to the public under the MIT license, please see LICENSE.txt. The project is an implementation of [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway's_Game_of_Life), and also provides many example initial configurations.

*PreEmptive Protection for iOS* integrates into the build process to produce obfuscated output. It is composed of two parts:

* [PreEmptive Protection for iOS - Control Flow](https://www.preemptive.com/products/ppios), or *PPiOS-ControlFlow* for short, is a tool for applying control-flow obfuscation to Objective-C iOS apps. For additional information about *PPiOS-ControlFlow* please contact [PreEmptive Solutions](https://www.preemptive.com/contact/contactus), or download a [free trial](https://www.preemptive.com/eval-request).
* [PreEmptive Protection for iOS - Rename](https://github.com/preemptive/ppios-rename) (available on GitHub), or *PPiOS-Rename* for short, is a tool for applying renaming obfuscation to Objective-C class, protocol, property, and methods names, in iOS apps. *PPiOS-Rename* is licensed under the GNU GPL v2, but commercial support is also available from [PreEmptive Solutions](https://www.preemptive.com/contact/contactus) via a commercial support agreement.

The documentation for [*PPiOS-ControlFlow*](https://www.preemptive.com/images/stories/ppios-controlflow/userguide.html) and [*PPiOS-Rename*](https://github.com/preemptive/PPiOS-Rename/blob/master/README.md) describe in detail how to integrate them into an existing Xcode project. Those steps have already been applied for this project, and it is ready to be loaded in Xcode.

The rest of this document details how to use this sample, describes the different ways to build the project, explains how the project was configured, and examines information contained in the build output.


## How to Use this Sample

To make best use of this sample, you will need:

* Xcode 8
* *PPiOS-ControlFlow* v2.5.0, or later (free trial or full version)
* *PPiOS-Rename* v1.1.0, or later (open source)
* [git](https://git-scm.com/) version control system

Once the project has been cloned, some configuration is required to get the project to build correctly with *PPiOS-ControlFlow* and *PPiOS-Rename*.

1. Clone the repository with `git`:

        git clone https://github.com/preemptive/PPiOS-Sample-Vie.git

2. Verify that *PPiOS-ControlFlow* is installed correctly:
    1. Verify that the Xcode integration is correct by executing the following command. This should show the version of *PPiOS-ControlFlow* that is installed.

            clang -control-flow=off --version
            # PPiOS-ControlFlow LLVM version 8.0.0 (ppios-controlflow-2.5.0+xxxxxxx)
            # ...

    2. Refer to the [documentation](https://www.preemptive.com/images/stories/ppios-controlflow/userguide.html) for *PPiOS-ControlFlow* about how to install *PPiOS-ControlFlow* in alternate Xcode installations.

3. Verify the installation path for *PPiOS-Rename*:
    1. In Xcode 8, go to the Project Navigator.
    2. Select the `Vie` project, expand the "project and targets list" if not already open, and select `Build and Analyze Vie`.
    3. Select Build Phases, and expand the `Analyze Binary` build phase.
    4. Verify that *PPiOS-Rename* is installed at `${HOME}/Downloads/PPiOS-Rename-v1.1.0`, or update the path accordingly.
    5. Verify the path similarly for the `Apply Renaming to Vie` target.
    6. If the path was updated, save these changes to your local copy of the repository with:

            git add -u
            git commit -m "Update PPiOS-Rename path"


## Building In Xcode

The fully obfuscated (control-flow and renaming) build takes a few steps:

1. In Xcode, select your preferred simulator device as the destination. Both iPhone and iPad are supported. Testing on a device will require a full version of *PPiOS-ControlFlow* and will require providing signing credentials.
2. Select the `Build and Analyze Vie` target, clean, and build.
3. Select the `Apply Renaming to Vie` target, and build.
4. Select the `Vie` target, clean, and build.
5. At this point you should be able to test the obfuscated app by selecting `Project` | `Run`.
6. In the game running in the simulator, tap `Quick Play` and tap the play button at the bottom. This will create a random initial layout and start the game, demonstrating basic functionality of the app.

To build with just renaming applied:

1. Select the `Unobfuscated Vie` target, clean, and build (since renaming has already been applied to the sources from above). This will create `Unobfuscated Vie`, named to avoid confusion with the version with control-flow obfuscation applied (although this app may or may not have renaming obfuscation applied, depending on whether the `Apply Renaming to Vie` target was built).
2. Test the app as desired.

To build with just control-flow obfuscation applied:

1. Revert the renaming changes (applied above) using `git` from a Terminal:

        cd PPiOS-Sample-Vie  # change directory into your cloned copy of the repository
        git checkout -- .    # this also replaces symbols.map with an empty map
        rm symbols.h

2. Select the `Vie` target, clean, and build.
3. Test the app as desired.

To build without either form of obfuscation:

1. Ensure any renaming changes have been reverted as above.
2. Select the `Unobfuscated Vie` target, clean, and build.
3. Test the app as desired.


## Configuration

Obfuscation for this project was configured as described in the documentation for *PPiOS-ControlFlow* and *PPiOS-Rename*.

The important bits are located in 3 places:

1. `CFLAGS` for `Vie` Target
2. *PPiOS-ControlFlow* Configuration
3. *PPiOS-Rename* Configuration

### `CFLAGS` for `Vie` Target

To find this configuration value:

1. Go to the Project Navigator, select the `Vie` target, select `Build Settings`.
2. Select `All` build settings, select `Combined`, and search for `cflags`.
3. Double-click the `Other C Flags` value.

This will show the options in a list:

    -ppconfig=ppconfig.yaml
    -ppios-rename-map=symbols.map
    -mllvm
    -debug-only=trace

`-ppconfig` specifies the name of the *PPiOS-ControlFlow* configuration file, `ppconfig.yaml`, which can be found in the root of the project. `-ppios-rename-map` specifies the name of the `symbols.map` file produced by *PPiOS-Rename* which allows the exclusion rules in `ppconfig.yaml` to be applied correctly. The pair of options, `-mllvm -debug-only=trace`, tells [`llvm`](http://llvm.org/) (part of `clang`) to emit additional information pertinent to obfuscation. See the `Build Log Details` section below.

### *PPiOS-ControlFlow* Configuration

The configuration file is [YAML](http://www.yaml.org/)-based, and reproduced here in full:

    control-flow:
      level: high
      branch-injection:
        block-percent: 50
      defaults:
        exclude:
          - "-[FWGameViewController calculateNextCycle]"
          - "-[FWGameViewController liveCellsGroupedByAgeFromGameMatrix:]"
          - "+[FWGameViewController ageGroupFromAge:]"
          - "-[FWGameViewController generateInitialCellsWithColumns:rows:percentageOfLiveCells:]"
          - "[FWRandomNumberGenerator "

The `level` set to `high` indicates that all four control-flow transforms are to be applied.

Specifying a `block-percent` of 50 indicates that only 50% of blocks (after block-injection) will have branch-injection applied. The build output below shows that only 39 of 77 blocks in `-[FWGameViewController setPattern:]` had branch-injection applied to them. Reducing the `block-percent` is one way to mitigate slowness caused by increased code complexity of obfuscated code. For this app this is likely unnecessary.

The excludes listed in the `exclude` section of `defaults` will apply to all of the transforms. The most processing-intensive parts of apps may need to be excluded from obfuscation for performance reasons. This app, which computes the next state of the game using a naive approach, will be slowed noticably unless `-[FWGameViewController calculateNextCycle]` is excluded (note that this exclusion does not affect similarly named method `-[FWGameViewController calculateNextCycle:]`). The next two exclusions are for methods that are called from within the nested for loops in `calculateNextCycle`. Since methods are obfuscated independently, longer methods called from within critical loops may also need to be excluded.

For this app these exclusions are probably sufficient. However, methods involved in creating the initial random game state are also excluded as examples: one additional method in `FWGameViewController` and all methods in `FWRandomNumberGenerator`.

It may be useful to see how obfuscation can affect performance.

1. Run the fully obfuscated app on the `iPad Pro (12.9 inch)` simulator device.
2. Tap `Quick Play`, tap the hamburger button in the upper-left corner, and tap `settings`.
3. Scroll down and tap the smallest size icon, which corresponds to the most cells.
4. Tap the "three dot" icon for the fastest speed.
5. Scroll up, tap the "X", tap `restart`, and tap the "play" button.

The app should run quickly, updating the game state at maybe 10-20 times a second.  It should be hard to tell one game state transition from the next.

1. Stop the app.
2. Comment out the `defaults` section in `ppconfig.yaml`, prefixing each line with a pound-sign `#` (octothorpe).
3. Clean, build, and run the app again (`Vie` target, renaming steps need not be run again beforehand).
4. Tap `Quick Play`, and tap the "play" button.

The app should be noticably slower, updating the game state at only 3-4 times a second.

This demonstration is not intended to say that obfuscation will make the performance of your app unacceptably slow. It will make it slower, but adjusting what obfuscation is applied to performance-critical parts of your app is an effective way to maintain a positive user experience.

### *PPiOS-Rename* Configuration

All of the configuration of *PPiOS-Rename* is done in the `Analyze Binary` Build Phase of the `Build and Analyze Vie` target, as command-line options to `ppios-rename`:

    PATH="${PATH}:${HOME}/Downloads/PPiOS-Rename-v1.1.0"
    [[ "${SDKROOT}" == *iPhoneSimulator*.sdk* ]] && sdk="${SDKROOT}" || sdk="${CORRESPONDING_SIMULATOR_SDK_DIR}"
    ppios-rename --analyze \
      --sdk-root "${sdk}" \
      -x FWSavedGamePickerViewController \
      -x FWPatternPickerViewController \
      -x FWAboutViewController \
      "${BUILT_PRODUCTS_DIR}/${EXECUTABLE_PATH}"

Only three options have been added to the default script prescribed by the *PPiOS-Rename* instructions.

This application uses `.xib` files, and the files are referenced by name in two ways:

1. Explicitly as string literals passed to `-[UIViewController initWithNibName:bundle:]` (`.xib` files are compiled into `.nib` files), and
2. Implicitly in calls to `-[UIViewController init]`.

Of the ten `.xib` files in the project, six are referenced by name explicitly in string literals. These require no change since *PPiOS-Rename* changes the names of classes, but not the names of the `.xib` files. One file, `FWLaunchScreen`, is referenced by name in the project file, and again requires no change.

The other three are `FWSavedGamePickerViewController`, `FWPatternPickerViewController`, and `FWAboutViewController`. When the filenames are not referenced explicitly, the resources are searched for at run-time based on the name of the class. Thus, these names must be excluded from renaming. The `-x` option is used here to exclude just the class names, rather than `-F` which would exclude the classes' members as well.


## Build Log Details

### *PPiOS-ControlFlow* Details

With trace output turned on as shown above, the build log will contain additional details. To see this additional information:

1. Go to the Report Navigator, and select `By Group`.
2. Expand the section for the `Vie` target, and select a `Build` report.
3. Select an Objective-C compilation step (`FWGameViewController.m` for example), and select the hamburger button to the right.

The text is composed of two parts separated by a horizontal line. Above the separator line are the details of the command issued by Xcode, and below are the details of what *PPiOS-ControlFlow* did:

		License successfully verified for 'Licensed P. User'
		Reading Configuration: /Users/ppios/Vie/ppconfig.yaml
		Using randomly-generated random seed: 0x9a4c8a6a8b5cf8a3a74729d01834ca4b
		Reading Rename Map: /Users/ppios/Vie/symbols.map
		...
		switch-obfuscation: Processing function '-[FWGameViewController setPattern:]' ('-[l644PETpKqHcPAkeLs4K setPattern:]')
		switch-obfuscation: Obfuscated 26 of 26 blocks for function '-[FWGameViewController setPattern:]' ('-[l644PETpKqHcPAkeLs4K setPattern:]') 
		block-injection: Processing function '-[FWGameViewController setPattern:]' ('-[l644PETpKqHcPAkeLs4K setPattern:]')
		branch-injection: Processing function '-[FWGameViewController setPattern:]' ('-[l644PETpKqHcPAkeLs4K setPattern:]')
		branch-injection: Obfuscated 39 of 77 blocks for function '-[FWGameViewController setPattern:]' ('-[l644PETpKqHcPAkeLs4K setPattern:]')
		opaque-predicates: Processing function '-[FWGameViewController setPattern:]' ('-[l644PETpKqHcPAkeLs4K setPattern:]')
		...
		switch-obfuscation: Skipped Function '-[FWGameViewController generateInitialCellsWithColumns:rows:percentageOfLiveCells:]' ('-[l644PETpKqHcPAkeLs4K z6ExcNGVQdluyaupKInfwTQ1P2I5m6J:rows:g17AIOO72KwJ4x53vbU2p:]') (Matched exclude rule '-[FWGameViewController generateInitialCellsWithColumns:rows:percentageOfLiveCells:]')
		block-injection: Skipped Function '-[FWGameViewController generateInitialCellsWithColumns:rows:percentageOfLiveCells:]' ('-[l644PETpKqHcPAkeLs4K z6ExcNGVQdluyaupKInfwTQ1P2I5m6J:rows:g17AIOO72KwJ4x53vbU2p:]') (Matched exclude rule '-[FWGameViewController generateInitialCellsWithColumns:rows:percentageOfLiveCells:]')
		...

Application of each control-flow transform on each method is shown in the text. Since the app has renaming applied, the original name is shown with the new name in parentheses. The `-ppios-rename-map` option allows the *PPiOS-ControlFlow* exclusion rules to be applied properly, as shown for `-[FWGameViewController generateInitialCellsWithColumns:rows:percentageOfLiveCells:]`.

For these two methods, the two components of the selectors, `setPattern` and `rows`, remain unrenamed. These names are automatically excluded to avoid producing conflicts in other code. These are two of the many symbols marked for exclusion when the SDK is analyzed in the `Analyze Binary` script. See the `Forbidden keywords` line in the *PPiOS-Rename* output below.

### *PPiOS-Rename* Details

*PPiOS-Rename* by default emits messages during the build, and these can be useful to understand what is going on from a high level. To see this information:

1. In the Report Navigator, the section for the `Build and Analyze Vie` target, and select a `Build` report.
2. Scroll to the bottom.

You should find a fair amount of text similar to the following:

		2017-03-02 09:40:28.600 ppios-rename[9251:16226029] Fetching symbols from CoreData model at path file:///Users/ppios/PPiOS-Sample-Vie/Vie/DataModel.xcdatamodeld/DataModel.xcdatamodel/contents
		2017-03-02 09:40:29.398 ppios-rename[9251:16226029] Processing external symbols from PhysicsKit...
		...
		2017-03-02 09:40:29.846 ppios-rename[9251:16226029] Processing external symbols from Security...
		2017-03-02 09:40:29.846 ppios-rename[9251:16226029] Processing internal symbols...
		2017-03-02 09:40:29.847 ppios-rename[9251:16226029] Adding @protocol FWAboutViewControllerDelegate
		2017-03-02 09:40:29.847 ppios-rename[9251:16226029] Adding @protocol FWColorTileDelegate
		...
		2017-03-02 09:40:29.853 ppios-rename[9251:16226029] Ignoring @protocol NSObject
		...
		2017-03-02 09:40:29.855 ppios-rename[9251:16226029] Adding @class FWAppDelegate
		2017-03-02 09:40:29.856 ppios-rename[9251:16226029] Adding @class FWGameViewController
		...
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] Generating symbol table...
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] Protocols = 20
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] Classes = 36
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] Categories = 3
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] Methods = 524
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] I-vars = 129
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] Filters = 372
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] Ignore symbol patterns = 5
		2017-03-02 09:40:29.872 ppios-rename[9251:16226029] Forbidden keywords = 86583
		2017-03-02 09:40:29.894 ppios-rename[9251:16226029] Done generating symbol table.
		2017-03-02 09:40:29.894 ppios-rename[9251:16226029] Generated unique symbols = 1043

*PPiOS-Rename* reports that it is processing symbols in Core Data model. Names used in the data model are used in what is written to persistent storage, and are thus automatically excluded from renaming to allow continued use of these stores across versions of the app.

All of the frameworks referenced directly or indirectly by the app are listed with `Processing external symbols from` as they are found. Afterward, `Processing internal symbols...` indicates that the symbols in the app are being read.

Next, each of the protocols, classes, and categories in or referenced by the app are examined, checking these identifiers with respect to the class and symbol exclusion rules.

Finally, a summary describing the results of the analysis process is printed. The number of protocols, classes, categories, methods, and ivars found in the app are printed. This is followed by `Filters = 372`, indicating the number of class include/exclude patterns that were applied, including exclude patterns for classes found in the SDK (which in this case is all of them). This is followed by `Ignore symbol patterns = 5`, indicating the number of symbol exclusion patterns that were applied, including two that are automatically added. This is followed by `Forbidden keywords = 86583`, indicating the number of non-class symbols found in the SDK, or names explicitly excluded by *PPiOS-Rename*. Last, the line `Generated unique symbols = 1049` indicates the number of symbols to be renamed.

It is expected that *PPiOS-Rename* will emit warnings about parsing some pieces of the SDK. These can be ignored.

> Note: You may see some variation from the numbers reported above. This may be caused by a number of factors including: iOS SDK version/build, and Xcode version.


Copyright 2017 PreEmptive Solutions, LLC

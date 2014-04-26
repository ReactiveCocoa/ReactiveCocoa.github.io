---

title: "Tutorial: Setup a new project with cocoapods"
author: "@coneko"
date: 2014-04-26 14:59 UTC
tags: cocoapods, tutorial

---

There are many different ways of setting up your project to use RAC, the following tutorial will lead you through the process of doing that using cocoapods for dependency management, specta for testing and expecta for matching.

Note: the recommended way of installing RAC is by adding it as a git submodule, not by using cocoapods.

1. Create a new project in Xcode

	Create a new project in Xcode like you normally would. Any kind of project is fine.

2. Create a podfile

	Cocoapods loads the project's configuration from a file called `Podfile` in the project's root directory.
	Go ahead and create it now, with the following contents:

		platform :osx, '10.9'

		pod 'ReactiveCocoa', '~> 2.2'

	Put `:ios` as your platform if the project you created is for iOS. You can specify the version of the platform as well, but keep in mind ReactiveCocoa requires OS X 10.7+ or iOS 5.0+.

3. Install the pods

	Use cocoapods to install ReactiveCocoa and create a workspace:

		$ pod install
	
	After the command has completed successfully, you will find a workspace with the same name as the project file in your project's root directory. The workspace will contain your project and a project that builds the dependencies installed through cocoapods, in this case ReactiveCocoa.
	
	Open the workspace and try to run your application now.

4. Add some ReactiveCocoa code

	Open the workspace if you haven't already, and edit the application delegate to add some code that uses ReactiveCocoa to check that everything works.

		#import "CNAppDelegate.h"

		#import <ReactiveCocoa/ReactiveCocoa.h>

		@implementation CNAppDelegate

		- (void)applicationDidFinishLaunching:(NSNotification *)aNotification
		{
			[[RACSignal return:@"Hello World"] subscribeNext:^(NSString *greeting) {
				NSLog(@"%@", greeting);
			}];
		}

		@end

	Run your application and you should see the greeting in the console.
	
5. Create a test target

	If your project template does not include a test target, add one now.

6. Add testing dependencies

	Add the testing and matching framework to the `Podfile`:
	
		target :TutorialSetupANewProjectWithCocoaPodsTests do
			pod 'Specta', '~> 0.2'
			pod 'Expecta', '~> 0.3'
		end
		
	Put the name of the unit testing target in your project instead of `:TutorialSetupANewProjectWithCocoaPodsTests`.
	
7. Create a spec

	Specta has a very different way of defining tests, so you can delete the test file the unit testing target template should have created for you, and create a new one with the following contents:
	
		#import <Specta/Specta.h>
		#define EXP_SHORTHAND
		#import <Expecta/Expecta.h>

		#import <ReactiveCocoa/ReactiveCocoa.h>

		SpecBegin(App)

		describe(@"A signal", ^{
			it(@"should send values", ^{
				id value = @"Hello";
				RACSignal *signal = [RACSignal return:value];
				expect([signal first]).to.equal(value);
			});
		});

		describe(@"A subscription", ^{
			it(@"should receive values", ^{
				id value = @"Hello";
				__block id receivedValue = nil;
				RACSignal *signal = [RACSignal return:value];

				[signal subscribeNext:^(id x) {
					receivedValue = x;
				}];

				expect(receivedValue).will.equal(value);
			});
		});

		SpecEnd

	You can define tests with the `it` function, and optionally group tests together with the `describe` function. `describe` can be nested multiple times.
	
	You can define assertions with `expect(value).to` for a normal assertion or `expect(value).will` for an asynchronous assertion.
	
	Try to run the tests now to see if it all works.
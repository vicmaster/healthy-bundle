#A Healthy Bundle

If you're writing Rails applications today, then you're probably enjoying Bundler when adding dependencies to your Rails applications. Bundler lets you declare your application's dependencies clearly and concisely, and it will figure out which available versions satisfy your requirements.

Version Requirements

Assuming that your dependencies are released gems, there are three ways to specify your dependencies:

### exact version
	gem 'nokogiri', '1.0.3'
	gem 'webrat', '0.3.1'

### pessimistic version
	gem 'nokogiri', '~> 1.0.3'
	gem 'webrat', '~> 0.3.1'

### any version
	gem 'nokogiri'
	gem 'webrat'
Let's take a look at some of the ups and downs of each approach.

##Exact Version

If you specify the exact version of every gem, then you'll never have unexpected breaks while upgrading. Running bundle is the same as running bundle update, and you'll always know exactly what's installed. You won't have to worry about updating capybara and getting a new version of rack that may affect your production environment.

It has some major downsides, though. You can only keep a tight lock on every gem if you also specify the exact version of every dependency, including your dependencies' dependencies, and so on. This means that you'll need to chase down a viable version of every gem in use in your application's bundle, even if you're not using it directly. It also means that you need to track when grand-dependencies change or are no longer required.

It means that Bundler can no longer do its job well. Every version you specify removes options from Bundler's resolution algorithm, which means you have to do all the work yourself. Let's look at a simple upgrade scenario: let's say you decide to upgrade from webrat 0.3.1 to webrat 0.4.0. You change your Gemfile:

	gem 'nokogiri', '1.0.3'
	gem 'webrat', '0.4.0'
And run bundle to install the new version.

However, this particular version of webrat depends on a newer version of nokogiri, so you'll have to bump nokogiri gem as well. If you have other dependencies pegged to a specific version of nokogiri, you're now privileged to play a fun game of dependency whack-a-mole. Common gems like rack and json are particularly problematic in this respect.

In summary, specifying exact versions gives you a lot of control and insight into dependencies in your applications, but it makes adding and updating dependencies difficult.

##Pessimistic Version

Specifying a pessimistic version tells Bundler that it's fine to upgrade a gem, as long as the gem version doesn't change too much. This works best in conjunction with gems which follow semantic versioning. With semantic versioning, you can tell Bundler to upgrade gems as long as the new versions only contain bug fixes by pegging a minor version. You can give Bundler more freedom by pegging a minor version, which will mean that only backwards-compatible changes are allowed.

Loose versions also have drawbacks, however. Although semantic versioning says that gems shouldn't break compatibility, there's no guarantee that somebody won't do so by mistake, or introduce an unrelated bug. You also can't depend on every gem following semantic versioning.

Looser versions also increase the number of potential changes which occur when you change your Gemfile, which makes it more likely that you'll accidentally break something when adding a new dependency.

By specifying pessimistic versions, you give up some of the control from exact versions, but adding and updating dependencies becomes much easier, and versions won't change to drastically without your knowledge.

##Versionless

You can also tell Bundler that you don't care at all by leaving the version out entirely. In this case, Bundler will install the latest versions of all your dependencies that work together. It's worth noting that you still get a reliable development environment, because Bundler stores the versions of all your dependencies separately in Gemfile.lock. This means that running bundle won't change the installed versions of any dependencies it doesn't need to.

Leaving the version out is less work initially, since you don't have to look up the version or add it to the file. It doesn't restrict Bundler at all, which means it can do the best job possible of finding versions that work together. It also makes it extremely easy to update all your gems, since everything will get the latest version from a bundle update.

However, without placing any restrictions on the version, you open yourself up to dramatic updates in your gems. Running a bundle update may result in backwards-incompatible changes to your dependencies, and the risk of introducing newer versions of gems with new bugs is higher.

##Be Purposeful

The benefits and drawbacks to these three approaches have led us to a combined approach that treats each gem specially.

Frameworks like Rails are especially likely to introduce breaks, even with patch updates, so locking such gems to an exact version is best. Upgrading your framework needs to be handled manually as a separate task, and locking to an exact version allows you to decide when to undertake that work.

Many gems have a strong history of backwards-compatibility, such as pg, thin, and debugger. Breakage from upgrading those gems is unlikely and will be spotted immediately, so specifying a version number is unnecessary.

Other gems frequently have backwards-incompatible changes but are reliable between minor or patch versions. Examples include rspec, factory_girl, and capybara. These gems are good candidates for pessimistic versions.

Using the correct version restriction for each gem expresses purpose and intention for those gems. Using an exact version represents a fragile dependency; using no version represents a safe and loose dependency that can be updated often. When updating or adding dependencies, use an intention-revealing git commit message that delivers meaningful information to the current and future team.

##Take Little Steps

You're going to need to upgrade your application's dependencies. Security vulnerabilities are discovered, new features arrive to increase productivity, and bugs are fixed. Interdependencies will force you to update more gems than you intend to, and each upgrade requires debugging and fixing incompatibilities in your application. Therefore, it's best to upgrade continually, in little steps.

If you tighten the reins too much on your dependencies, it will discourage you from updating frequently. Find a balance that works for your application, your team, and your dependencies, but make sure that you give yourself enough room for continuous updates.


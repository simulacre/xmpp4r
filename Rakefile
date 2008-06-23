#!/usr/bin/ruby -w

require 'rake'
require "rake/clean"
require 'rake/testtask'
require 'rake/gempackagetask'
require 'rake/rdoctask'
require 'find'

$:.unshift 'lib'
require "xmpp4r"

PKG_NAME  = 'xmpp4r'
AUTHORS   = ['Lucas Nussbaum', 'Stephan Maka']
EMAIL     = "xmpp4r-devel@gna.org"
HOMEPAGE  = "http://home.gna.org/xmpp4r/"
SUMMARY   = "XMPP4R is an XMPP/Jabber library for Ruby."

##############################################################################
# DEFAULT TASK
##############################################################################

# The task that will run when a simple 'rake' command is issued.
# Default to running the test suite as that's a nice safe command
# we should run frequently.
task :default => [:test]

##############################################################################
# TESTING TASKS
##############################################################################

Rake::TestTask.new do |t|
  t.libs << "test"
  t.test_files = ['test/ts_xmpp4r.rb']
end

##############################################################################
# DOCUMENTATION TASKS
##############################################################################

# RDOC
#######
Rake::RDocTask.new do |rd|
  f = []
  require 'find'
  Find.find('lib/') do |file|
    # Skip hidden files (.svn/ directories and Vim swapfiles)
    if file.split(/\//).last =~ /^\./
      Find.prune
    else
      f << file if not FileTest.directory?(file)
    end
  end
  f.delete('lib/xmpp4r.rb')
  # hack to document the Jabber module properly
  f.unshift('lib/xmpp4r.rb')
  rd.rdoc_files.include(f)
  rd.options << '--all'
  rd.options << '--fileboxes'
  rd.options << '--diagram'
  rd.options << '--inline-source'
  rd.options << '--line-numbers'
  rd.rdoc_dir = 'rdoc'
end

# RCOV
#######

# Conditional require rcov/rcovtask if present
begin
  require 'rcov/rcovtask'

  Rcov::RcovTask.new do |t|
    t.test_files = ['test/ts_xmpp4r.rb']
    t.output_dir = "coverage"
  end
rescue Object
end

# DOT GRAPH
############
desc "Generate requires graph"
task :gen_requires_graph do
  sh %{cd tools; ./gen_requires.bash}
end


##############################################################################
# SYNTAX CHECKING
##############################################################################

desc "Check syntax of all Ruby files."
task :check_syntax do
  `find . -name "*.rb" |xargs -n1 ruby -c |grep -v "Syntax OK"`
  puts "* Done"
end

##############################################################################
# PACKAGING & INSTALLATION
##############################################################################

# What files/dirs should 'rake clean' remove?
CLEAN.include ["*.gem", "pkg", "rdoc", "coverage", "tools/*.png"]

# The file list used to package tarballs, gems, and for generating the xmpp4r.gemspec.
PKG_FILES = %w( LICENSE COPYING README.rdoc README_ruby19.txt CHANGELOG Rakefile setup.rb xmpp4r.gemspec ) + Dir["{lib,test,data,tools}/**/*"]

spec = Gem::Specification.new do |s|
  s.name = PKG_NAME
  s.version = Jabber::XMPP4R_VERSION
  s.authors = AUTHORS
  s.email = EMAIL
  s.homepage = HOMEPAGE
  s.rubyforge_project = PKG_NAME
  s.summary = SUMMARY
  s.description = s.summary
  s.platform = Gem::Platform::RUBY
  s.require_path = 'lib'
  s.executables = []
  s.files = PKG_FILES
  s.test_files = []

  # rdoc
  s.has_rdoc = true
  s.extra_rdoc_files = %w( README.rdoc README_ruby19.txt CHANGELOG LICENSE COPYING )
  s.rdoc_options = ["--quiet", "--title", "xmpp4r documentation", "--opname", "index.html", "--line-numbers", "--main", "README.rdoc", "--inline-source"]

  s.required_ruby_version = ">= 1.8.4"

end

Rake::GemPackageTask.new(spec) do |pkg|
  pkg.gem_spec = spec
  pkg.need_tar = true
end

# also keep the gemspec up to date each time we package a tarball or gem
task :package => ['gem:update_gemspec']
task :gem => ['gem:update_gemspec']

namespace :gem do

  desc "Run :package and install the .gem locally"
  task :install => [:update_gemspec, :package] do
    sh %{sudo gem install --local pkg/#{PKG_NAME}-#{Jabber::XMPP4R_VERSION}.gem --no-rdoc --no-ri}
  end

  desc "Run :clean and uninstall the .gem"
  task :uninstall => :clean do
    sh %{sudo gem uninstall #{PKG_NAME}}
  end

  # Thanks to the Merb project for this code.
  desc "Update Github Gemspec"
  task :update_gemspec do
    skip_fields = %w(new_platform original_platform date)
    integer_fields = %w(specification_version)

    result = "# WARNING : RAKE AUTO-GENERATED FILE.  DO NOT MANUALLY EDIT!\n"
    result << "# RUN : 'rake gem:update_gemspec'\n\n"
    result << "Gem::Specification.new do |s|\n"
    spec.instance_variables.each do |ivar|
      value = spec.instance_variable_get(ivar)
      name  = ivar.split("@").last
      next if skip_fields.include?(name) || value.nil? || value == "" || (value.respond_to?(:empty?) && value.empty?)
      if name == "dependencies"
        value.each do |d|
          dep, *ver = d.to_s.split(" ")
          result <<  "  s.add_dependency #{dep.inspect}, #{ver.join(" ").inspect.gsub(/[()]/, "")}\n"
        end
      else
        case value
        when Array
          value =  name != "files" ? value.inspect : value.sort.uniq.inspect.split(",").join(",\n")
        when String
          value = value.to_i if integer_fields.include?(name)
          value = value.inspect
        else
          value = value.to_s.inspect
        end
        result << "  s.#{name} = #{value}\n"
      end
    end
    result << "end"
    File.open(File.join(File.dirname(__FILE__), "#{spec.name}.gemspec"), "w"){|f| f << result}
  end

end


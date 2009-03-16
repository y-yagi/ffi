require 'rubygems'

USE_RAKE_COMPILER = (RUBY_VERSION =~ /1\.9.*/ || RUBY_PLATFORM =~ /java/) ? false : true
if USE_RAKE_COMPILER
  gem 'rake-compiler', '>=0.3.0'
  require 'rake/extensiontask'
end

require 'date'
require 'fileutils'
require 'rbconfig'

begin
  require 'bones'
  Bones.setup
rescue LoadError
  begin
    load 'tasks/setup.rb'
  rescue LoadError
    raise RuntimeError, '### please install the "bones" gem ###'
  end
end

LIBEXT = Config::CONFIG['host_os'].downcase =~ /darwin/ ? "dylib" : "so"
GMAKE = Config::CONFIG['host_os'].downcase =~ /bsd/ ? "gmake" : "make"
LIBTEST = "build/libtest.#{LIBEXT}"
BUILD_DIR = "build/#{RUBY_VERSION}"

# Project general information
PROJ.name = 'ffi'
PROJ.authors = 'Wayne Meissner'
PROJ.email = 'wmeissner@gmail.com'
PROJ.url = 'http://kenai.com/projects/ruby-ffi'
PROJ.version = '0.3.0'
PROJ.rubyforge.name = 'ffi'

PROJ.readme_file = 'README.rdoc'

# Annoucement
PROJ.ann.paragraphs
PROJ.ann.email[:from] = 'wmeissner@gmail.com'
PROJ.ann.email[:to] << 'dev@ruby-ffi.kenai.com' << 'users@ruby-ffi.kenai.com'
PROJ.ann.email[:server] = 'smtp.gmail.com'

# Gem specifications
PROJ.gem.need_tar = false
PROJ.gem.files = %w(LICENSE README.rdoc Rakefile) + Dir.glob("{ext,lib,nbproject,samples,spec}/**/*")
PROJ.gem.platform = Gem::Platform::RUBY
PROJ.gem.extensions = %w(ext/ffi_c/extconf.rb gen/Rakefile)

# RDoc
PROJ.rdoc.exclude << '^ext/'

TEST_DEPS = [ LIBTEST ]
if RUBY_PLATFORM == "java"
  desc "Run all specs"
  task :specs => TEST_DEPS do
    sh %{#{Gem.ruby} -S spec #{Dir["spec/ffi/*_spec.rb"].join(" ")} -fs --color}
  end
  desc "Run rubinius specs"
  task :rbxspecs => TEST_DEPS do
    sh %{#{Gem.ruby} -S spec #{Dir["spec/ffi/rbx/*_spec.rb"].join(" ")} -fs --color}
  end
else
  TEST_DEPS.unshift :compile
  desc "Run all specs"
  task :specs => TEST_DEPS do
    ENV["MRI_FFI"] = "1"
    sh %{#{Gem.ruby} -Ilib -I#{BUILD_DIR} -S spec #{Dir["spec/ffi/*_spec.rb"].join(" ")} -fs --color}
  end
  desc "Run rubinius specs"
  task :rbxspecs => TEST_DEPS do
    ENV["MRI_FFI"] = "1"
    sh %{#{Gem.ruby} -Ilib -I#{BUILD_DIR} -S spec #{Dir["spec/ffi/rbx/*_spec.rb"].join(" ")} -fs --color}
  end
end

desc "Build all packages"
task :package => 'gem:package'

desc "Install the gem locally"
task :install => 'gem:install'

unless USE_RAKE_COMPILER
  file "#{BUILD_DIR}/Makefile" do
    FileUtils.mkdir_p(BUILD_DIR) unless File.directory?(BUILD_DIR)
    sh %{cd "#{BUILD_DIR}" && #{Gem.ruby} #{Dir.pwd}/ext/ffi_c/extconf.rb}
  end
  desc "Compile the native module"
  task :compile => "#{BUILD_DIR}/Makefile" do
    sh %{cd "#{BUILD_DIR}"; make}
  end
end
desc "Clean all built files"
task :clean do
  FileUtils.rm_rf("build")
  FileUtils.rm_rf("conftest.dSYM")
  FileUtils.rm_f(Dir["pkg/*.gem"])
end
task "build/libtest.#{LIBEXT}" do
  sh %{#{GMAKE} -f libtest/GNUmakefile}
end
desc "Build test helper lib"
task :libtest => "build/libtest.#{LIBEXT}"
desc "Test the extension"
task :test => [ :specs, :rbxspecs ] do
end
namespace :bench do
  ITER = ENV['ITER'] ? ENV['ITER'].to_i : 100000
  bench_libs = "-Ilib -I#{BUILD_DIR}" unless RUBY_PLATFORM == "java"
  bench_files = Dir["bench/bench_*.rb"].reject { |f| f == "bench_helper.rb" }
  bench_files.each do |bench|
    task File.basename(bench, ".rb")[6..-1] => TEST_DEPS do
      sh %{#{Gem.ruby} #{bench_libs} #{bench} #{ITER}}
    end
  end
  task :all => TEST_DEPS do
    bench_files.each do |bench|
      sh %{#{Gem.ruby} #{bench_libs} #{bench}}
    end
  end
end

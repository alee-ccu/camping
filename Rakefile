$:.unshift 'extras'

begin
  require 'rake/dsl_definition'
  require 'rake/alt_system'
rescue LoadError
else
  begin
    if defined?(Rake::DeprecatedObjectDSL)
      Rake::DeprecatedObjectDSL.class_eval do
        private_instance_methods(false).each do |meth|
          remove_method meth
        end
      end
    end
  rescue Exception
  end
end

require 'rake'
require 'rake/clean'
require 'rake/testtask'
require 'tempfile'
require 'open3'

require File.expand_path('../constants', __FILE__)

CLEAN.include ['**/.*.sw?', '*.gem', '.config', 'test/test.log', '.*.pt']

task :default => :check

task :rubygems_docs do
  require 'rubygems/doc_manager'
  
  def spec.installation_path; '.' end
  def spec.full_gem_path;     '.' end
  manager = Gem::DocManager.new(spec)
  manager.generate_rdoc
end

desc "Packages Camping."
task :package => :clean

## Tests
Rake::TestTask.new(:test) do |t|
  t.libs << "test"
  t.test_files = FileList['test/app_*.rb']
end

## Diff
desc "Compare camping and camping-unabridged"
task :diff do
  require 'parser/current'
  require 'unparser'
  require 'pp'
  u = Tempfile.new('unabridged')
  m = Tempfile.new('mural')
  
  usexp = Parser::CurrentRuby.parse(File.read("lib/camping-unabridged.rb"))
  msexp = Parser::CurrentRuby.parse(File.read("lib/camping.rb"))

  u << Unparser.unparse(usexp)
  m << Unparser.unparse(msexp)
  
  u.flush
  m.flush
  
  sh "diff -u #{u.path} #{m.path} | less"
  
  u.delete
  m.delete
end

error = false

## Check
task :check => ["test", "check:valid", "check:equal", "check:size", "check:lines", "check:exit"]
namespace :check do

  desc "Check source code validity"
  task :valid do
    sh "ruby -c lib/camping.rb"
  end

  desc "Check equality between mural and unabridged"
  task :equal do
    require 'ruby_parser'
    u = RubyParser.new.parse(File.read("lib/camping-unabridged.rb"))
    m = RubyParser.new.parse(File.read("lib/camping.rb"))
    
    u.reject! do |sexp|
      sexp.is_a?(Sexp) and sexp[1] == s(:gvar, :$LOADED_FEATURES)
    end
    
    unless u == m
      STDERR.puts "camping.rb and camping-unabridged.rb are not synchronized."
      error = true
    end
  end

  SIZE_LIMIT = 4096
  desc "Compare camping sizes to unabridged"
  task :size do
    FileList["lib/camping*.rb"].each do |path|
      s = File.size(path)
      puts "%21s : % 6d % 4d%" % [File.basename(path), s, (100 * s / SIZE_LIMIT)]
    end
    if File.size("lib/camping.rb") > SIZE_LIMIT
      STDERR.puts "lib/camping.rb: file is too big (> #{SIZE_LIMIT})"
      error = true
    end
  end

  desc "Verify that line lenght doesn't exceed 80 chars for camping.rb"
  task :lines do
    i = 1
    File.open("lib/camping.rb").each_line do |line|
      if line.size > 81 # 1 added for \n
        error = true
        STDERR.puts "lib/camping.rb:#{i}: line too long (#{line[-10..-1].inspect})"
      end
      i += 1
    end
  end
  
  task :exit do
    exit 1 if error
  end

end

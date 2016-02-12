#!/usr/bin/env rake
# Builds pkginfo files for munki using ERB templates
# cribbed heavily from: https://code.google.com/p/munki/wiki/ManagingPrintersWithMunki

# define the printers you want in the printers.yml file

require 'erb'
require 'yaml'

class PrinterSettings
  attr_accessor :settings

  def initialize(file)
    @settings = YAML.load_file(file)
  end

  def get_binding
    return binding()
  end
end


task :default => ["build"]

desc "Build pkginfo files from printer description YAML files in printers folder"
task :build do

  FileList["printers/*.yml"].each do |printer|

    name = printer.pathmap("%n") # just the filename, no path, no extension

    printer_settings = PrinterSettings.new(printer) # read in the YAML file, create a binding object

    FileList["templates/*.erb"].each do |template| # process through all the .erb files 

      script_out = template.pathmap("%n") # just the filename, no path, no extension
        
      template_erb = ERB.new(File.read(template), nil, '<>') # 

      File.open("workdir/#{name}_#{script_out}", "w+") do |script|
        script.write(template_erb.result(printer_settings.get_binding))
      end
    end

    noquotes_desc = printer_settings.settings["description"].gsub("'", "") # nuke single quotes as they break the following shell commands and escaping them escapes me at the moment.

    # build a list of required packages to send to makepkginfo (usually drivers)
    required_string = "";
    unless printer_settings.settings["requires"].nil?
      printer_settings.settings["requires"].each do |requirement|
        required_string << " --requires '#{requirement}' "
      end
    end

    sh "makepkginfo --nopkg --name='#{name}' \
        --displayname='#{printer_settings.settings["displayname"]}' \
        --pkgvers='#{printer_settings.settings["version"]}' \
        --description='#{noquotes_desc}' \
        #{required_string} \
        --installcheck_script='workdir/#{name}_installcheck' \
        --postinstall_script='workdir/#{name}_postinstall' \
        --uninstall_script='workdir/#{name}_uninstall' \
        > 'pkgsinfo/#{name}.pkginfo'"

  end
end


desc "remove all generated files"
task :clean do
  sh "rm -f workdir/*"
  sh "rm -f pkgsinfo/*"
end


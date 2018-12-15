require 'semantic'
require 'erb'
require 'json'
require 'yaml'

NO_AVAILABLE_VERSION_ERROR

NAMESPACE = 'jmatsu'

desc 'Release the Orb'
task :release do
  next_version = Semantic::Version.new(ENV.fetch('RELEASE_VERSION'))
  term = get_term(current_version: current_version, release_version: next_version)

  exec_command!("circleci orb publish promote #{namespace_and_orb_name}@dev:#{ENV.fetch('SNAPSHOT_LABEL')} #{term} --token #{ENV.fetch('CIRCLECI_TOKEN')}")
end

desc 'Deploy the Orb to staging'
task :staging do
  exec_command!("circleci orb publish #{namespace_and_orb_name}@dev:#{ENV.fetch('STAGING_LABEL')} --token #{ENV.fetch('CIRCLECI_TOKEN')}")
end

desc 'Validate the Orb'
task :validate do
  orb_yml = ENV['ORB_YML'] || 'orb.yml'

  raise "#{orb_yml} is not found" if File.exist?(orb_yml)

  exec_command!("circleci orb validate #{orb_yml}")
end

private

# @param [String] file_path a path to a file
def expand_aliases(file_path)
  YAML.load(YAML.load_file(file_path).to_json).to_yaml
end

def namespace_and_orb_name
  "#{NAMESPACE}/#{ENV.fetch('ORB_NAME')}"
end

def current_version
  response = exec_command!("circleci orb info #{namespace_and_orb_name} --token #{ENV.fetch('CIRCLECI_TOKEN')}")

  raise 'failed to ' if $? != 0

  version = response.lines.find {|l| l.include?("Latest")}.match(/([0-9]+\.[0-9]+\.[0-9]+)/).captures.first

  raise 'Could not retrieve the version' if version.nil?

  Semantic::Version.new(version)
end

def get_term(current_version:, release_version:)
  if current_version.increment!(:major) == release_version
    :major
  elsif current_version.increment!(:minor) == release_version
    :minor
  elsif current_version.increment!(:patch) == release_version
    :patch
  else
    raise 'could not determine the term of the version'
  end
end

def exec_command!(command)
  response = `#{command}`

  raise "failed to execute #{command} : #{response}" if $? != 0

  response
end
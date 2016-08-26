require "middleman-gh-pages"
require "json"
require "bundler"

desc "Generates all the static resources necessary to run the site."
tasks = ["plugins_core", "plugins_external", "grab_dangerfiles", "search_plugin_json", "getting_started_docs"]
task generate: tasks.map { |task| "generator:" + task } do
  # Your Gemfile.lock tends to get put out of sync after some of the commands.
  puts "Shippit"
end

desc "Things that generate JSON"
namespace :generator do
  desc "Generates a JSON file that represents the core plugins documentation"
  task :plugins_core do
    # Make sure we have a folder for json_data, it's not in git.
    Dir.mkdir("static/json_data") unless Dir.exist?("static/json_data")

    # Grab the Danger gem, pull out the file paths for the core plugins
    danger_gem = Gem::Specification.find_by_name "danger"
    danger_core_plugins = Dir.glob(danger_gem.gem_dir + "/lib/danger/danger_core/plugins/*.rb")

    # Document them, move them into a nice JSON file
    output = `bundle exec danger plugins json #{danger_core_plugins.join(" ")}`
    abort("Could not generate the core plugin metadata") if output.empty?
    File.write("static/json_data/core.json", output)
    puts "Generated core API metadata"
  end

  desc "Generates a JSON file that represents the external plugins documentation"
  task :plugins_external do
    plugins = JSON.parse(File.read("plugins.json"))
    # plugins = ["danger-xcodebuild"]
    plugin_objects = []
    plugin_search_objects = []

    plugins.each do |plugin|
      puts "Generating docs for #{plugin}"
      # Generate the Website's plugin doc, by passing in the gem names
      plugin_json = `bundle exec danger plugins json #{plugin}`
      next unless $?.success?
      plugin_objects << JSON.parse(plugin_json)
    end

    File.write("static/json_data/plugins.json", plugin_objects.flatten.to_json)
    puts "Generated plugin metadata"

    plugins_file_path = File.join File.dirname(__FILE__), "plugins-search-generated.json"
    gem_metadata = plugin_objects.flatten.map { |p| p["gem_metadata"] }

    File.write(plugins_file_path, { plugins: gem_metadata }.to_json)
    puts "Generated search metadata for `danger search`."
  end

  desc "Generates a JSON file that represents the external plugins documentation"
  task :grab_dangerfiles do
    # Grab our Dangerfile plugins list
    dangerfile_repos = JSON.parse(File.read("example_oss_dangerfiles.json"))
    dangerfile_repos.each do |repo|
      require "open-uri"
      require "pygments"
      branch = repo.include?("#") ? repo.split("#").last : "master"
      repo = repo.split("#").first

      dangerfile = open("https://raw.githubusercontent.com/#{repo}/#{branch}/Dangerfile").read

      path = "static/source/dangerfiles/#{repo.tr('/', '_')}.html"
      html = Pygments.highlight(dangerfile, lexer: "ruby", options: { encoding: "utf-8" })

      File.write(path, html)
    end
    puts "Downloaded Dangerfiles to `static/source/dangerfiles`"
  end

  desc "Generate the website plugin search JSON file, this is different from the danger gem search - as one gem can have multiple plugins"
  task :search_plugin_json do
    plugins = JSON.parse(File.read("static/json_data/plugins.json"))
    plugin_search_metadata = plugins.map do |plugin|
      {
        name: plugin["name"],
        gem: plugin["gem"],
        body: plugin["body_md"],
        instance: plugin["instance_name"],
        tags: plugin["tags"],
        see: plugin["see"]
      }
    end

    File.write("static/json_data/plugin_search.json", plugin_search_metadata.to_json)
    puts "Generated search JSON for inline search"
  end

  desc "Generate the getting started guides metadata from Danger"
  task :getting_started_docs do
    `bundle exec danger systems ci_docs > static/json_data/ci_docs.json`
    puts "Generated getting started CI documentation"
  end
end

desc "Runs the site locally"
task :serve do
  puts "Running locally at http://localhost:4567"
  sh "open http://localhost:4567"
  Dir.chdir("static") do
    sh "bundle exec middleman server"
  end
end

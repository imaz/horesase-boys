task :default => :build

desc "Build dist/meigens.json"
task :build do
  require "yaml"
  require "json"

  dest_path = "dist/meigens.json"
  puts "Generating %s ..." % dest_path

  records = Dir["data/*"].map do |path|
    data = YAML.load_file(path)
    body_path = File.join("body", File.basename(path, ".yml") + ".txt")
    body = File.exist?(body_path) ? File.read(body_path) : nil
    data["body"] = body

    data
  end
  records = records.sort_by {|record| record["id"] }
  num_with_body = records.inject(0) do |sum, record|
    sum + (record["body"] ? 1 : 0)
  end
  puts "%d of %d (%0.1f%%) have body text" % [
    num_with_body,
    records.size,
    num_with_body.to_f / records.size.to_f * 100
  ]

  bytes_written = File.write(dest_path, JSON.generate(records))
  puts "%d bytes written" % bytes_written
end

desc "Fetch metadata"
task :fetch do
  require_relative "lib/jigokuno"
  require "psych"
  require "yaml"
  require "active_support/core_ext"

  horesasu = Jigokuno::Misawa.new

  horesasu.each { |meigen|
    yaml_path = File.join(File.dirname(__FILE__), "data", "#{meigen[:id]}.yml")
    break if File.exist? yaml_path

    puts "[%3s] %s / %s / %s" % [:id, :title, :character, :image].map { |attr| meigen[attr] }

    yaml = YAML.dump(meigen.stringify_keys)
    File.write(yaml_path, yaml)

    sleep 2
  }
end

desc "Upload to Github Download section"
task :upload => :build do
  require "github_downloads"
  uploader = GithubDownloads::Uploader.new
  uploader.authorize

  uploader.upload_file("meigens.json", "Latest build", "dist/meigens.json")
end

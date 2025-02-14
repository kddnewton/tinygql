require "rake/testtask"

Rake::TestTask.new(:test) do |t|
  t.libs << "test"
  t.libs << "lib"
  t.warning = true
  t.test_files = FileList["test/**/*_test.rb"]
end

FILES = [ "lib/tinygql/nodes.rb", "lib/tinygql/visitors.rb" ]

task :test => FILES

task default: :test

def extract_nodes source
  require "psych"
  require "erb"

  info = Psych.load_file source
  node = Struct.new(:name, :parent, :fields) do
    def has_children?; fields.length > 0; end

    def human_name
      name.gsub(/([A-Z]+)([A-Z][a-z])/,'\1_\2').
        gsub(/([a-z\d])([A-Z])/,'\1_\2').
        tr("-", "_").
        downcase
    end

    def children
      buf = ["ary = []"]

      fields.each do |field|
        case field.type
        when "list"
          x = "ary.concat(#{field.name})"
          if field.nullable?
            x << " if #{field.name}"
          end
          buf << x
        when "node"
          x = "ary << #{field.name}"
          if field.nullable?
            x << " if #{field.name}"
          end
          buf << x
        end
      end

      buf << "ary"
      buf.join("; ")
    end
  end

  field = Struct.new :_name, :type do
    def name
      _name.sub(/\?$/, '')
    end

    def list?; type == "list"; end
    def node?; type == "node"; end
    def visitable?; list? || node?; end

    def nullable?
      _name.end_with?("?")
    end
  end
  info["nodes"].map { |n|
    node.new(n["name"], n["parent"] || "Node", (n["fields"] || []).map { |f|
      name = f
      type = "node"
      if Hash === f
        (name, type) = *f.to_a.first
      end
      field.new name, type
    })
  }
end

file "lib/tinygql/nodes.rb" => "lib/tinygql/nodes.yml" do |t|
  nodes = extract_nodes t.source
  erb = ERB.new File.read("lib/tinygql/nodes.rb.erb"), trim_mode: "-"
  File.binwrite t.name, erb.result(binding)
end

file "lib/tinygql/visitors.rb" => "lib/tinygql/nodes.yml" do |t|
  nodes = extract_nodes t.source
  erb = ERB.new File.read("lib/tinygql/visitors.rb.erb"), trim_mode: "-"
  File.binwrite t.name, erb.result(binding)
end

require "rake/clean"
CLOBBER.append(FILES)

desc 'Running Jekyll with --server --auto opition'
task :dev do
	  system('jekyll --server --auto')
end

desc "Given a title as an argument, create a new post file"
task :write, [:title, :category] do |t, args|
	  filename = "#{Time.now.strftime('%Y-%m-%d')}-#{args.title.gsub(/\s/, '_').downcase}.markdown"
		    path = File.join("_posts", filename)
	    if File.exist? path; raise RuntimeError.new("Won't clobber #{path}"); end
	      File.open(path, 'w') do |file|
		          file.write <<-EOS
			  ---
				  layout: post
			  category: #{args.category}
				  title: #{args.title}
				  date: #{Time.now.strftime('%Y-%m-%d %k:%M:%S')}
				  ---
				  EOS
			      end
	          puts "Now open #{path} in an editor."
end

namespace :juicer do
	  desc 'Merges stylesheets'
	      task :css => :"juicer:js" do
		            sh 'juicer merge --force _site/style/master.css'
			        end
	        desc 'Merges JavaScripts'
		    task :js do
			          sh  'juicer merge -i --force _site/js/master.js'
				      end
end


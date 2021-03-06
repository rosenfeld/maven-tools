#-*- mode: ruby -*-

gemspec

# we need only the metadata - no extenions, deps, repos, plugins !!
model.build.extensions.clear
model.build.plugins.clear
model.dependencies.clear
model.repositories.clear

parent 'org.sonatype.oss:oss-parent', '7'

packaging 'jar'
group_id 'de.saumya.mojo'

# let ruby-maven build snapshots only
spec_version = model.version.to_s
version spec_version + ( ENV['RELEASE'] ?  '' : '-SNAPSHOT')

u = model.url.sub!( /^https?:\/\//, '' ) if model.url
source_control( model.url,
                :connection => "scm:git:git://#{u}.git",
                :developer_connection => "scm:git:ssh://git@#{u}.git" )

profile( :install ) do

  gemspec

  build do
    default_goal :install
  end

  plugin 'de.saumya.mojo:gem-maven-plugin', '${jruby.plugins.version}' do
 #jruby_plugin( :gem ) do
    execute_goal :initialize
  end
end

profile( :test ) do

  gemspec

  build do
    default_goal :test

    plugin( 'de.saumya.mojo:minitest-maven-plugin', '${jruby.plugins.version}', 
            :minispecDirectory =>"spec/*spec.rb" ) do
      execute_goals(:spec)
    end

    plugin( 'de.saumya.mojo:rspec-maven-plugin', '${jruby.plugins.version}',
            :specSourceDirectory=>"rspec" ) do
      execute_goals(:test)
    end

    # Gemfile.lock is just a convenience but a library should work with
    # any allowed version. clean will force a bundle install
    plugin(:clean, '2.5', 
           :filesets => [ { :directory => './',
                            :includes => ['Gemfile.lock'] } ] )
    
  end
end

profile :push do

  build do
    default_goal :deploy

    plugin :deploy, :skip => true
    
    plugin 'de.saumya.mojo:gem-maven-plugin', '${jruby.plugins.version}' do
    
      # push gem to rubygems on deploy
      execute_goals( :push, :id => 'gem push',
                     :gem => "${project.build.directory}/maven-tools-#{spec_version}.gem" )
    
    end
  end
end

profile 'sonatype-oss-release' do

  build do
    default_goal :deploy

    plugin :deploy, :skip => false
  end
end

plugin 'de.saumya.mojo:gem-maven-plugin', '${jruby.plugins.version}' do

  # build the gem along with the jar
  execute_goals( :package, :id => 'gem build',
                 :phase => :package,
                 :gemspec => 'maven-tools.gemspec' )
end

# just lock down the versions
properties( 'jruby.plugins.version' => '1.0.0-rc4',
            'jruby.version' => '1.7.4',
            # overwrite via cli -Djruby.versions=1.6.7
            # no more 1.5.6 since it gives problem with backports gem
            'jruby.versions' => ['1.6.8','1.7.4'].join(','),
            # overwrite via cli -Djruby.modes=2.0
            'jruby.modes' => '1.8,1.9,2.0'
           )

properties 'tesla.dump.pom' => 'pom.xml'

# add the ruby files to jar
resource do
  directory '${project.basedir}/lib'
end

# TODO find a better way
# include dependent gems as well
# manually add transitive dependencies of virtus as well
resource do
  directory '${project.build.directory}/rubygems/gems/virtus-0.5.5/lib'
  includes [ '**/*' ]
end
resource do
  directory '${project.build.directory}/rubygems/gems/backports-3.3.5/lib'
  includes [ '**/*' ]
end
resource do
  directory '${project.build.directory}/rubygems/gems/descendants_tracker-0.0.3/lib'
  includes [ '**/*' ]
end

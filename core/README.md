= Merb Upload

This plugin for Merb provides a simple and extremely flexible way to upload files.

=== Getting Started

At the moment you are going to have to grab it here from github and install it yourself

Add it as a dependency to your config/init.rb
    
    dependency 'merb_upload'

=== Simple Uploaders

You can create your own uploaders for maximum flexibility. You can use them any way you like from your models or controllers. You're not constrained to uploading files submitted by the user. The aim of these uploaders is to give you the same flexibility you enjoy when you roll your own upload solution, while taking away all the fiddly stuff by using a sane set of conventions.

First, Generate an uploader:

    merb-gen uploader avatar
    
You'll have an uploader now in app/uploaders/avatar_uploader.rb, it'll look something like this:
  
    class AvatarUploader < Merb::Upload::Uploader

      # Choose what kind of storage to use for this uploader
      storage :file

    end

This is actually already pretty much useable. merb_uploads separates between the 'store' and the 'cache'. The store is for permanent storage, and the cache is a temporary place to put files to facilitate manipulation. By default, merb_upload will always use the cache, if you do not want/need it (for performance reasons) you can switch it off in your init.rb like this:

    Merb::Plugins.config[:merb_upload][:use_cache] = false
    
==== Storing things in the cache

By default the cache directory is ./public/upload/tmp. You might want to change this if you do not want temporary uploads to be publicly accessible.

Every file stored in the cache has a cache_id, the cache_id is of the format YYYYMMDD-HHMM-PID-RND. By default, cached files will be in a subdirectory with that name. This should ensure file descriptors do not run out, that there are no collisions between files and that it is relatively simple to parse out the date of their creation so they can easily be swept when no longer needed.

Once you instantiate an uploader with an identifier, like this:

    uploader = AvatarUploader.new('myfile.png')

You can store a file in the cache like this:

    cache_id = uploader.cache!(File.open('path/to/file'))

You only need the cache_id when the file should be retrieved much later (such as in a later request).

    uploader.retrieve_from_cache!(cache_id)

==== Storing things in the storage

The storage is where files get stored on a permanent basis. It could be the local filesystem, or maybe something like Amazon S3. merb-upload is agnostic about what storage you would like to use.

You can change the store via the 'storage' class method.

Unlike in the cache, files are uniquely identified by their *identifier* in the store (there's no equivalent to cache_id).

Files are stored like this:

    uploader.store!(file)
    
And retrieved like this:
    
    result = uploader.retrieve_from_store!(file)

What +result+ is depends on the storage engine, however by convention, result should have a +url+ method, which points at the location of the file.

 
=== Model Uploaders

This kind of level of control is often unnecessary. More often than not, we simply want to attach a file to a model and be done with it. This is where model uploaders come in. Assuming the +users+ table has a column named +avatar+ we could do something like this:

    # uploader:
    class AvatarUploader < Merb::Upload::ModelUploader
      # Choose what kind of storage to use for this uploader
      storage :file
    end
    
    # model:
    class User < ActiveRecord::Base
      attach :avatar, AvatarUploader
    end
    
Now you can upload file like this:

    # puts the file into the cache
    @user = User.new :name => params[:name], :file => params[:file]
    # puts the file into the store
    @user.save!
    
It may be unneecessary or even undesirable to store the entire filename, instead it is often enough to store the extension, and have the file be uniquely identified by the ID of the model for example. To set this up, you can set the +:extension_only+ option to +attach+:

    # uploader:
    class AvatarUploader < Merb::Upload::ModelUploader
      # Choose what kind of storage to use for this uploader
      storage :file
      
      def filename
        "#{model.id}.#{identifier}"
      end
    end

    # model:
    class User < ActiveRecord::Base
      attach :avatar, AvatarUploader, :extension_only => true
    end

==== DataMapper
      
ModelUploaders are compatible with DataMapper's types, you can do this:

    class User
      include DataMapper::Resource
      
      property :avatar, AvatarUploader
      
    end
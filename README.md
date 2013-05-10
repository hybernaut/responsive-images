# Responsive Images

ResponsiveImages is a simple library that handles responsive images on both the server and client side for Rails projects. It provides a set of helper methods that automatically load the appropriate image for whatever device a visitor is using.

The library depends on Carrierwave and lets you use the versions created in your uploader to serve device-appropriate images. 

Unlike traditional responsive javascript libraries, ResponsiveImages has a server-side component that will detect the visitor's device and create the actual image tag src based on the most appropriate image size. The helper will also add a few data attributes that allow the javascript file to handle responsive images on the front-end.

## Installation

Add this line to your application's Gemfile:

    gem 'responsive_images'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install responsive_images
    

## Configuration

The Responsive Image gem is easy to setup and use. Please note that it is currently requires Rails and [Carrierwave](https://github.com/jnicklas/carrierwave). To get started, you should setup or identify your carrierwave versions, here are some examples:

    class MyUploader < CarrierWave::Uploader::Base
      ...
    
      # Mobile version
      version :mobile do
        process :resize_to_fit => [300, 300]
      end
  
      # Tablet version
      version :tablet do
        process :resize_to_fit => [600, 600]
      end
  
      # Desktop version
      version :desktop do
        process :resize_to_fit => [900, 900]
      end
    
      ...
    end
  
You can add as many as you'd like. 

#### Initializer
In order to configure the gem, you'll need to add an initializer in app/initializers/responsive_images.rb

    ResponsiveImages.configure do |config|
      # Set the default version for image. If you leave it at :default then it will use
      # the original size, i.e., image.url or you can use a specific version
      config.default = :default  
      # Add some custom sizes that you'll generate with Carrierwave. You can make as many
      # as you want. But you'll need to configure mobvious for any custom ones.
      config.sizes = {
        mobile: :mobile,            # carrierwave version size
        tablet: :tablet,            # another carrierwave version size
        desktop: :desktop           # and one more version...
      }
    end

Note that :mobile, :tablet and :desktop should be replaced with whatever carrierwave version you created. You can add more version but you'll need to configure mobvious to identify these screen sizes. ResponsiveImages does not currently support this but it is a feature that will eventually be added.

## Usage

ResponsiveImages is incredibly easy to use. There are currently two primary helpers


### Responsive Image Tag

The `responsive_image_tag` is meant replace the `image_tag`. You can pass any arguments, just like you would with an `image_tag`. Here is a basic exmpale:

    # basic usage
    = responsive_image_tag @post.image
    # => <img class="responsive" src="path/to/image.jpg", data-desktop-src="path/to/desktop_image.jpg" data-tablet-src="path/to/tablet_image.jpg" data-mobile-src="path/to/mobile_image.jpg" /> 

The helper method will determine what size is most appropriate for the `src` attribute. Rather than load a default size and then swap them out, the ResponsiveImage gem actually detects the user device and uses the appropriate image for the `src` attribute.

TODO: the 'responsive' class is added to the emitted image tag in order to trigger the javascript as described below. It might make sense to make this configurable or more easily overridden.

You can also pass custom sizes to the helper method. If you have a specific model or page that use's different image, then pass those Carrierwave versions through the :sizes parameter:
  
    # custom versions
    = responsive_image_tag @post.image, :sizes => { :tablet => :post_tablet, :mobile => :post_mobile }
    # => <img class="responsive" src="path/to/image.jpg", data-desktop-src="path/to/desktop_image.jpg" data-tablet-src="path/to/post_tablet_image.jpg" data-mobile-src="path/to/post_mobile_image.jpg" /> 
  
These custom versions will overwrite the default versions from our configuration. This is incredibly useful for pages or models that require a different set of sizes. For example, a page hero/banner might be much larger than most content images and requires custom sizes.


### Responsive Background Image

ResponsiveImages also includes a helper for background images. You can easily call this with:
    
    # basic usage
    = responsive_background_image @post.image
    # => style="url(path/to/image.jpg)" data-desktop-src="path/to/image.jpg" data-tablet-src="path/to/tablet_image.jpg" data-mobile-src="path/to/mobile_image.jpg"

This will automatically create the attributes for your `<div/>` or whatever tag you add a background image too.

You can also pass custom version to the helper method with:
  
    # custom versions
    = responsive_background_image @post.image, :sizes => { :'mobile' => :post_small, :tablet => :post_tablet }
    # => style="url(path/to/image.jpg)" data-desktop-src="path/to/image.jpg" data-tablet-src="path/to/post_tablet_image.jpg" data-mobile-src="path/to/post_mobile_image.jpg"


### Client-side JavaScript

The ResponsiveImages helpers will allow you to construct your responsive image tags and even select the most appropriate size to load initially but if the browser window sizes changes, we'll need some client-side javascript to adjust the image src. The responsive-images.js is included in this gem but needs to be added to your javascript bundle, in application.js add:

    //= require responsive-images
    
The responsive-images.js is based on jQuery, so you'll need jQuery loaded.

Now we need to instantiate the plugin. This is quite simple:
    
    // setup our responsive images
    $('img.responsive').responsive_images({
      mobile_size: 480,
      tablet_size: 768,
      desktop_size: 980,
    });
    
Unlike the helper methods, with the JS library, you define breakpoints that will tell the plugin to find the appropriate size and resize it to the appropriate data-attribute. 
For example, if you're breakpoint is mobile_size then the plugin will look for a data-mobile-src attribute. 

  
### Mobvious

ResponsiveImages uses [Mobvious](https://github.com/jistr/mobvious) for device detection. Check out [the gem](https://github.com/jistr/mobvious) for more details. You can use Mobvious to detect the user device by simply calling:
  
    return request.env['mobvious.device_type']
    # => desktop (mobile or tablet)


## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

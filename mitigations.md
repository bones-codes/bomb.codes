---
layout: default
permalink: mitigations
---

# Mitigations

To effectively mitigate decompression bomb attacks, a layered defense is the best approach. Implementing various checks and restrictions for application and processes will ensure that the attack is stopped before any lasting damage can occur.

<hr/>
## <a name="layer1"></a>Defense Layer 1

Client-side checks should never be relied upon for security. Server-side checks should be performed to validate:
1. File format is expected for context
2. Upload file size does not exceed the maximum limit 

<hr/>
## <a name="layer2"></a>Defense Layer 2

Limit the amount of resources available to the process and its children. This should be done at both the OS and application level.

For Linux platforms, cgroups can and should be used to limit both CPU and memory usage.

In Python resource limits can be configured via the `resource` module's `setrlimit` and `RLIMIT*` directives:

{% highlight python %}
import resource
rsrc = resource.RLIMIT_DATA
resource.setrlimit(rsrc, (1024000, hard)) # limit to 1MB
{% endhighlight %}

Ruby's `Process` module has similar `RLIMIT` directives

<hr/>
## <a name="layer3"></a>Defense Layer 3

In addition, filetype-specific mitigations should also be implemented depending on the context of the compression in use. 

<div class="btn-group">
  <a class="btn btn-default btn-ghost" href="#archives">Archives</a>
  <a class="btn btn-default btn-ghost" href="#images">Images</a>
  <a class="btn btn-default btn-ghost" href="#http">HTTP</a>
</div>
&nbsp;

### <a name="archives"></a>Archives

Restrict output file size and number of extracted files to ensure the total doesn't exceed the maximum limit. An exception should be thrown if either of these limits are reached.

{% highlight python %}
import zlib
def decompress(data, maxsize=1024000):
    dec = zlib.decompressobj()
    data = dec.decompress(data, maxsize)
    if dec.unconsumed_tail:
        raise ValueError("Possible bomb")
    del dec
    return data
{% endhighlight %}

Some filetypes have headers that report the output file size (ZIP, Gzip, etc.). These headers should be checked, but never solely relied upon since it isn't difficult to modify these values.

### <a name="images"></a>Images

Workers should be employed for process intensive tasks, such as image modifications. This will ensure that the application isn't knocked out due to a task such as a size increase.

In addition, default image library size limitations should be changed to reflect safe values that the process can handle in most circumstances. For example, libpng default size limitations are 1,000,000 by 1,000,000 pixels. Unless you're NASA and have the power to host and process such massive images, you should probably change these values to something a bit more sane. The default libpng size limitations can be modified via the `png_set_user_limits` method.

Prior to processing an image, dimensions should be programmatically checked. This can be accomplished via filetype headers. Various languages have libraries/modules that implement this check. For example, in Python this can be done using PIL's \texttt{Image} module:

{% highlight python %}
from PIL import Image
im = Image.open(image_filename)
width, height = im.size
# Check image dimensions
if (width < MAX_IMAGE_WIDTH) and (height < MAX_IMAGE_HEIGHT): 
    # do stuff
{% endhighlight %}

These values are extremly easy to change, so never rely solely upon the reported dimensions. Be sure to have both [layer 1](#layer1) and [layer 2](#layer2) defenses implemented.

When implementing layer 2 defenses for a libpng application, limits can be placed on memory consumption and ancillary chunks via the `png_set_chunk_malloc_max` and `png_set_chunk_cache_max` methods.

### <a name="http"></a>HTTP

Layer 2 defense implementation for Apache should make use of the various `RLimit*` directives (`RLimitCPU`, `RLimitMEM`, `RLimitNPROC`). For Nginx, the `worker_rlimit_core`, `worker_rlimit_nofile`, and `worker_processes` directives can be used.

Request sizes should be limited. Both Apache (`LimitRequestBody`) and Nginx (`client_max_body_size`) have directives to support this.

In addition, the accepted compression ratio should also be modified. If not, an attacker could simply edit the compression ratio to reduce the bomb to an accepted size. Apache supports this via `mod_deflate`'s `DeflateInflateRatioLimit`, `DeflateInflateRatioBurst`, and `DeflateWindowSize` directives.  


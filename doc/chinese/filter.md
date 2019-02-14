# Filter Plugin
Filter is the most important and basic plug-in used for filtering a 2d image.

## Invert

```python
from imagepy.core.engine import Filter

class Invert(Filter):
    title = 'Invert Demo'
    note = ['all', 'auto_msk', 'auto_snap']

    def run(self, ips, snap, img, para = None): 
        return 255-snap
```

Invert plugin, `note` indicates that the plug-in supports ROI and Undo operation. We can return the filter result in `run` function. About `snap` and `img`, `img` is the current image, if the `auto_snap` is added in `note`，the framework would copy the `img` to `snap` befor `run` function, as many filter operation need a buffer.  Besides, undo operation and ROI support must rely on `snap`.

![14](http://idoc.imagepy.org/demoplugin/13.png)

<div align=center>Invert</div><br>



## Gaussian

```python
from imagepy.core.engine import Filter
from scipy.ndimage import gaussian_filter

class Gaussian(Filter):
    title = 'Gaussian Demo'
    note = ['all', 'auto_msk', 'auto_snap','preview']
    para = {'sigma':2}
    view = [(float, 'sigma', (0,30), 1,  'sigma', 'pix')]
    
    def run(self, ips, snap, img, para = None):
        gaussian_filter(snap, para['sigma'], output=img)
```

Gaussian plugin，`note` indicates that the plug-in supports all types, `roi` and `undo` operation. It also supports the `preview` feature. `para` and `view` configure a float type  parameter `sigma`. In function `run`，the `scipy.ndimage.gaussian_filter` is called to filter the `snap`, and fill output `img`. If there is no output parameter in the function, return the result from `run`, then ImagePy would help us to fill `img` by the result.


![14](http://idoc.imagepy.org/demoplugin/14.png)

<div align=center>Gaussian</div><br>



## Filter operating mechanism

**note:** 
`note` option is the behavior control identifier, to control the flow of plugin execution. 
**i.e.:** Let the framework perform type compatibility detection, if not satisfied, it will be automatic ended. Set channel and sequence support settings. More : preview, ROI supports and other ones.

1. `all`: Plugin supports all types.
2. `8-bit`: Plugin supports unsigned 8 bit
3. `16-bit`: Plugin supports unsigned 16 bit
4. `int`: Plugin supports 32-bit, 64-bit integers
5. `rgb`: Plugin supports 3 channels, 24-bit color
6. `float`: Plugin supports 32-bit, 64-bit floating point.

------

7. `not_channel`: Do not automatically traverse the channel when processing multiple channels (by default, each channel will be processed in turn)

8. `not_slice`: When processing image sequences, do not ask `if you want to batch process` (users are asked by default)

9. `req_roi`: Whether there must be a ROI to be able to handle

---

10. `auto_snap`: Whether the framework is required to automatically buffer the current image on the processor

11. `auto_msk`: Whether to automatically support ROI (**must be effective with `auto_snap`**, the principle is to use `snap` to restore pixels outside the ROI)

12. `preview`: Whether to support preview (view results in real time), to adjust parameters

13. `2int`: Whether to convert data below `int32` to `int32` and then process it (for example, avoid `8-bit` operation overflow)

14. `2float`: Whether to automatically convert data below float32 to float32 and then process it during operation (some operations require precision)

    

**para, view:** Parameter dictionary, for specific usage, see [start](doc/start.md).

**run:** 

1. `ips`: Image wrapper class, usually does not need to be operated in `filter` 
2. `snap`: as the `auto_snap` is added in `note`, before `run`, the framework imagepy will copy the opened image to `snap` (as much as possible to deal with `snap`, and assign the results to `img`)
3. `img`: opened image, the result is assigned to `img`, or `return` results, assigned `img` by imagepy, and complete the interface refresh

**load:** 

`def load(self, ips)` is executed at first，if the `False` is returned with `return` , the plugin will be ended. Default return is `True`, if neccesary, it can be overloaded for a series of condition checks. If it is not satisfied, the `IPy.alert` will popup prompt, and return`False` 

**preview:**

`def preview(self, ips, para)` is executed during preview, the opened image is processed with calling `run` by default, and can be overloaded if necessary.
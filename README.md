# THIS PROJECT IS ARCHIVED   
Intel will not provide or guarantee development of or support for this project, including but not limited to, maintenance, bug fixes, new releases or updates.  
Patches to this project are no longer accepted by Intel.  
 If you have an ongoing need to use this project, are interested in independently developing it, or would like to maintain patches for the community, please create your own fork of the project.  
  
<a href="https://scan.coverity.com/projects/nneruntimeopenvino_v2">
  <img alt="Coverity Scan Build Status"
       src="https://scan.coverity.com/projects/32042/badge.svg"/>
</a>

# NNERuntimeOpenVINO
Intel's Unreal Engine OpenVINO plugin for NNE.

## Device & Model Support
The plugin supports all three NNE interfaces:
1. INNERuntimeCPU
2. INNERuntimeGPU
3. INNERuntimeNPU 

In cases where there are multiple GPUs, it's possible to override the default selection using a configuration setting `MultiGpuPreference` found in `Config\DefaultNNERuntimeOpenVINO.ini`. Integrated GPU will always be index 0 if present.

OpenVINO supports inference modes not present in NNE. To simplify NNE compatibility, it is currently not possible to select modes such as "AUTO", "HETERO", or "MULTI".

Supported model formats are:
1. OpenVINO IR (*.xml + *.bin)
2. ONNX (*.onnx)

While OpenVINO supports additional formats not listed here, they must be converted to either IR or ONNX for runtime use by the plugin. OpenVINO provides Python support for converting models. The scripts for performing this are included in the standalone distribution: https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/download.html

```
import openvino as ov
model = ov.convert_model("<input_path_to_model>")
ov.save_model(model, "<output_path_to_model>")
```

For best results, please use the IR format. The IR format is optimized for OpenVINO and the devices it supports. Depending on the model and device, you can expect to see up to a 10x performance improvement using IR.

ONNX model import is provided through the NNEEditor module, which is only required for Editor builds. OpenVINO IR import is handled by the Editor component of this plugin. IR models must be imported as a pair of files with matching names and .xml, .bin extensions. Both model formats will be stored as NNEModelData assets containing all model data.

Models are read and compiled at runtime when the ModelInstance is created.

## Platform Support
Windows and Linux are supported. For a full list of supported OS versions, please refer to: https://docs.openvino.ai/2025/about-openvino/release-notes-openvino/system-requirements.html

Prebuilt libraries can be found in:

`Binaries\openvino\<platform>`

The Linux libraries will need to be unzipped before building. This ensures symlinks are preserved.

If the prebuilt libraries don't meet your needs, you may build OpenVINO from source and replace the ones provided. This will require a rebuild of the Plugin as the symbols are resolved at compile time. Linux shared libraries are versioned so make sure to preserve the symlinks when replacing those files.
* Source: https://github.com/openvinotoolkit/openvino
* Build Guide: https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/build.md

### Drivers

#### Windows
Windows Update should automatically download the necessary drivers for NPU/GPU. If needed, you may either download those separately or using the Intel Driver & Support Assistant. The NPU driver does not currently come bundled with an installer.
* NPU Driver: https://www.intel.com/content/www/us/en/download/794734/intel-npu-driver-windows.html
* GPU Driver: https://www.intel.com/content/www/us/en/download/785597/intel-arc-iris-xe-graphics-windows.html
* Driver & Support Assistant: https://www.intel.com/content/www/us/en/support/detect.html

#### Linux
Depending on the distro and build, you may or may not have the necessary drivers pre-installed. Please refer to your distro's documentation for how to obtain these.
* NPU Driver: https://github.com/intel/linux-npu-driver
* GPU Driver: https://www.intel.com/content/www/us/en/support/articles/000005520/graphics.html

## Selective Device Build
The total distributable size in Release is around 100MB on Windows. If you do not intend to use a given interface it's possible to reduce this size by excluding the dynamic libraries for those interfaces.

The device libraries can be found in the following location:

`Binaries\openvino\<platform>\<build_type>`

Device specific dynamic libraries are named as follows:
1. openvino_intel_cpu_plugin
2. openvino_intel_gpu_plugin
3. openvino_intel_npu_plugin

Simply delete the unwanted device libraries from the folder. The plugin build script will detect if a device library is present and selectively enable that interface. Please note that if you have already built your project and go back to remove one of these libraries, the build script cache will still attempt to search for it. In this case you either need to invalidate the plugin build script or rebuild.

To rebuild just the plugin, you can use the following command:

`
Engine\Build\BatchFiles\RunUAT.bat BuildPlugin -plugin=MyProject\Plugins\NNERuntimeOpenVINO\NNERuntimeOpenVINO.uplugin -package=PackageDir\NNERuntimeOpenVINO -targetplatforms=Win64 -clean
`

## Support
Please refer to OpenVINO documentation for anything else not covered here: https://docs.openvino.ai/2025/index.html

## Special Thanks
Nico Ranieri of Epic Games for all his help in getting this released.

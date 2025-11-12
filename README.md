# WebGPULogger
WebGPULogger is a logger for WGSL shaders.  
Print and debug strings, u32, array, and vectors.   
ASCII chars supported only.  
Published under MIT License  

Note: There are 2 reserved integers that are used for markers inside the logger so you won't be able to print those. By default the logger is using -1 and -2 though you can modify them and reserve other values.
``` 
const INTSEQ_START_MARK = -1 >>> 0;  
const INTSEQ_END_MARK = -2 >>> 0;  
```

## Example

### WGSL Shader
```
// Logging: make sure you log only for a specific workitem/global_id
// if you don't the last workitem/global_id will overwrite the log
// so if you have multiple dimensions you need to specify all of them
enable_log(global_id.x == 0); // log only the 1st workitem/thread

// print scalar
var test: u32 = 10;   
console.log("scalar:", test);
   
// print array element
var test = array<u32, 4>(1,2,3,4);
console.log("arr element:", test[2]);
   
// print subarray from 1st to 3rd element
var test = array<u32, 4>(1,2,3,4);
console.log("subarray:", test[1:3]);
```

### JavaScript
```
// 1. import the logger module
import { WebGPULogger } from "./webgpu_logger.js";
```

```
// 2. create a logger
let logger = new WebGPULogger();
```

```
// 3. set your layout entries to the logger right after you create them:
// your layout entries
let layoutEntries = [
	{binding: 0, ...},
	{binding: 1, ...},
	{binding: 2, ...},
	];
logger.setBindGroupLayoutEntries(layoutEntries);
// you can now create your bind group layout
let bindGroupLayout = device.createBindGroupLayout({entries: layoutEntries});
```

```
// 4. pass your shader and the previous layout entries to get a log shader
let shader = "..."; // your WGSL shader
let logShader = logger.createLogShader(shader, layoutEntries);
// now you can create your shader module
const shaderModule = device.createShaderModule({code: logShader});
```

```
// 5. set the bind gorup entries to the logger right after you create them
// your bind entries
const bindGroupEntries = [
	{binding: 0, ...},
	{binding: 1, ...},
	{binding: 2, ...}
];
logger.setBindGroupEntries(bindGroupEntries);
// you can now create the bind group
const bindGroup = device.createBindGroup({
	label: 'transformBindGroup',
	layout: bindGroupLayout,
	entries: bindGroupEntries,
});
```

```	
// 6. set the commandEncoder
logger.setCommandEncoder(commandEncoder);
```

```
// 7. after submitting the job you can retrieve and print the log
device.queue.submit([commandEncoder.finish()]);
let log = await logger.getLog();
let lines = log.split("\n");
for(let line of lines)
	console.log(line);
```
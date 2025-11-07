# WebGPULogger
WebGPULogger is a logger for WGSL shaders.  
Print and debug strings, i32, u32, array, and vectors.   
ASCII chars supported only.

Shader:  
```
// Inside WGSL
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

JavaScript:  
```
import { WebGPULogger } from "./webgpu_logger.js";

let adapter = await nav.gpu.requestAdapter();
let device = await adapter.requestDevice();

// 1. create a logger
let logger=new WebGPULogger();

// 2. add the debug layout entry to your layout entries
let layoutEntries = [
	{binding: 0, visibility: GPUShaderStage.COMPUTE, buffer: { type: "storage"}}
	];
logger.addDebugBindLayoutEntry(layoutEntries);
device.createBindGroupLayout({entries: layoutEntries});

// 3. add the debug shader to your shader
let shader = "..."; // your shader
let debugShader = logger.createDebugShader(shader, layoutEntries);
// this is now the shaderModule to create your compute pipeline
const shaderModule = device.createShaderModule({code: debugShader});

// 4. add the debug buffer bind entry to your entries
const bindEntries = [
	{binding: 0, resource: {buffer: buffer}}
];
logger.addDebugBindEntry(bindGroupEntries);

// 5. set the commandEncoder
logger.setCommandEncoder(commandEncoder);

// 6. after submitting the job you can retrieve and print the log
device.queue.submit([commandEncoder.finish()]);
let log = await logger.getLog();
let lines = log.split("\n");
for(let line of lines)
	console.log(line);
```
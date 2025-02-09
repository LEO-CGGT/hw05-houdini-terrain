Houdini Terrain Generation - The Icy Mountain of Sleeping Giant
==================================
**University of Pennsylvania, CIS 566: Procedural Graphics, Project 4, Haoquan Liang**   
# Overview
In this project, we are exploring the powerful capabilities of Houdini in terrain generation. We will be experimenting different nodes and functions to create interesting-looking terrains with assets scattered on it.     
I wouldn't be able to get my result without [The Complete A-Z Terrain Handbook](https://www.sidefx.com/tutorials/the-complete-a-z-terrain-handbook/) by [Nikola Damjanov](https://www.sidefx.com/profile/damjanmx/). I learned a variety of heightfield nodes from this tutorial, and I followed its ideas in `08: ICY BIOME HF Breakdown` to get the snow-layered look in my terrain. I also followed `09: ICY BIOME COPs Texture Breakdown` to apply complex shading to the terrain.   

 ![overview](/img/overview.jpg)   
 ![overview2](/img/overview2.jpg)

# Features
* Complex snowy-mountain themed terrain with 40 nodes
* Snow-themed conifer trees generated by L-System, scattered naturally on the terrain (avoiding icy lakes, mountaintop, and giant human head)
* HDA widget that allows the user to change tree number/size/complexity(L-system iteration), and height of the mountains
* Advanced terrain shading with cop2net. 

# Details
### Terrain
 ![terrain](/img/terrain.jpg)   
 Terrain generation can be broken down into 6 steps.   
  ![terrain steps](/img/terrain-steps.png)   
 **1. Shape**   
 I want the terrain to have mainly visible four mountains. So I chose to scatter 4 tubes on a grid, and then project it to the heightfield. However, that will make the mountains look very uniform and pointy, so applying some blurs, noises, and distorts to make it look more natural.    
 **2. Noise**   
 The other parts of the terrain will still look very uniform. We layer Simplex and Worley noise on the terrain, as well as add some terrace on the mountain to make it look more like a mountain terrain.    
 **3. Erosion**   
 With the basic shape of the terrain done, we apply erosions to give the mountains and the terrain a more layered look so it looks like it's covered by thick snows.   
 **4. Terrace**   
 The slope of the mountains still looks flat. We add more terraces to it, and erode it so that it looks more like layers of rocks with some parts being covered by snow.    
 **5. Blend Giant Head**   
 The giant head mesh is found on [Sketchfab](https://sketchfab.com/3d-models/portrait-bust-of-augustus-marble-sculpture-5a7952aad74c4913a4f3a082132073c6). We only want to blend in this after applying the layered noises and erosions. Otherwise, all these noises will make the giant head unrecognizable. However, we still want to apply a small amount of noise to the mesh so that it can blend with the terrain better. After that, simply move it to the desired position and use `heightfield_project` to blend it into the terrain.       
 **6. Slump**   
Whereas the `Heightfield Erode` node simulate erosion by iterating over animation frames, `Heightfield Slump` calculates and applies the effect of slumping all at once. This is the final step to add "layers" to the slopes on the terrain.   

### Tree Generation and Scattering
 ![tree](/img/tree.png)   
 ![tree-scatter](/img/tree-scattering.jpg)   
Please refer to [this](https://github.com/LEO-CGGT/hw03-l-systems) for how the trees are generated.    
The L-system grammar itself is identical, I simply removed the gifts on the tree, and instead of shading the branches and leaves green, I used a `Attribute Wrangle` to blend white and green by its distance to the center of the tree. Since the trees will be very small on the terrain, 10 iterations is more than enough to make it look good enough.    
For the scattering, one approach is to use `Heightfield Mask by Feature` to select the parts of the terrain that can have trees. In my case, the icy lake and the mountaintop shoudn't have any trees, and I don't want the giant's face to be covered by trees. Instead of tuning the numbers to get the mask, I use `Heightfield Paint` to directly paint on the areas I want the trees to be scattered. After that, I use `copy to point` to instance all the trees.    

### HDA Widget
This part is designed to mimic professional packaging, in which the artists directly modify the attributes with a simple UI instead of going through all the nodes inside.    
 ![hda](/img/HDA-Widget.png)   
There are 4 attributes that can be modified on the HDA:    
1. Tree number (above 3000 is not recommended -- it will be very slow)
2. Tree size   
3. Tree complexity (L-system iteration number)
4. Mountain height (will take a very long time to process all the noises/erosion/slump again.
### Advanced Shading
 ![texture](/img/texture.png)   
The shading is done by creating a texture with `cop2net`. I highly recommend watching the [terrain handbook tutorial](https://www.sidefx.com/tutorials/the-complete-a-z-terrain-handbook/) as it goes the reasons behind each texture.    
In genral, all the terrain generation nodes will give us a masked terrain called layer. Each layer is the result of applying certain nodes. We use the slope layer to generate the snow texture (since snows stay on flat ground and flows on slope). We use the height layer to generate the "value" texture (higher is brighter) as well as the edge texture, since sudden height change usually means it's an edge. We use the flow layer to generate the thick snow layer on the slopes, and use the extra masked layer to generate the ridge lines. With all the above textures, we composite them together, adding some noises to generate the final texture.     

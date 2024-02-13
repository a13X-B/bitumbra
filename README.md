# bitumbra
many lights 2d shadows library for love framework

![image](https://github.com/a13X-B/bitumbra/assets/53251919/a5aa4456-da96-4a1f-a33a-771c421569f6)

requirements: opengl3 with array textures and instancing support(which technically should be a part of any gl3 capable system)

NOTE: functionality is still being added but already existing API is very unlikely to change

## bitumbra API:
```lua
local bt = require"bitumbra"
```
- `bt.newShadowMap(width, height)` - creates a new shadow map that you can render to/sample from(advanced, coming soon)
  - `width` and `height` are shadowmap dimensions
- `bt.newOcclusionMesh(max_edges)` - creates a new occlusion mesh that represents shadow casting geometry
  -  `max_edges` is the maximum amount of edges the mesh can store, 10000 by default
- `bt.newLightsArray()` - creates an array containing up to 128 lights that will cast shadows or can be drawn sampling the shadowmap

## lights array API:
```lua
local lights = bt.newLightsArray()
lights:push(x,y,radius, r,g,b)
lights:set(i, x,y,radius, r,g,b)
local x,y = lights:pop()
local n = lights.count
```
- `push(x,y,radius, r,g,b)` - pushes new light into the array, increases lights count
  - `x,y` - light position
  - `radius` - length at which the light falls to zero
  - `r,g,b` - red, green, and blue components of the light in the range of [0,1]
- `set(i, x,y,radius, r,g,b)` - sets
  - `i` - index of the light to set
  - `x,y,radius, r,g,b` - see `push`
- `pop()` - returns `x`, `y`, `radius`, and `r`,`g`,`b` of the latest light and removes it from the list
- `get(i)` - returns `x`, `y`, `radius`, and `r`,`g`,`b` of the light at index `i`
- `count` - a number of currently set lights in the array
- `draw(shadowmap)` - draws lights with respect of shadows stored in the shadowmap

## occlusion mesh API:
```lua
local geo = bt.newOcclusionMesh()
local circle = bt.newOcclusionMesh()
geo:addEdge(ax,ay, bx,by)
circle:addCircle(x, y, r)
geo:applyTransform(x, y, angle, sx, sy, ox, oy, kx, ky)
geo:addOcclusionMesh(circle)
```
- `addEdge(ax,ay, bx,by)` - adds an edge(straight line segment from point A to point B) to occlusion mesh
  - `ax,ay bx,by` - x and y positions of both ends of an edge segment
- `setEdge(i, ax,ay, bx,by)` - changes the existing edge
  - `i` - the index of the edge to change
- `addCircle(x, y, r, seg)` - adds a few edges shaped as a circle
  - `x, y` - center of the circle
  - `r` - circle radius
  - `seg` - number of segments that will form the circle(13 by default)
- `addOcclusionMesh(mesh)` - adds an existing occlusion mesh to the current mesh
  - `mesh` - an already existing occlusion mesh
  - useful when you want to combine a few meshes for the final effect
- `applyTransform(x, y, angle, sx, sy, ox, oy, kx, ky)` - applies the standard love transformation to every edge of the mesh
  - `x,y` - translation
  - `angle` - rotation angle in radians
  - `sx, sy` - scale
  - `ox, oy` - origin point, useful if you want to rotate a mesh around an arbitrary point
  - `kx, ky` - shear, I honestly have no idea who uses those but they're here for consistency with love API
  - can also accept love's very own Transform object(love.math.newTransform()) instead
- `edge_count` - current edge count of the mesh

## shadowmap API:
```lua
local sm = bt.newShadowMap(love.graphics.getDimensions())
```
- `sm.render(mesh, lights)` - renders occlusion `mesh` for every one of `lights` to the shadowmap

## example
```lua
local bu = require("bitumbra")
local g = love.graphics
local w, h = g.getDimensions()

local sm = bu.newShadowMap(g.getDimensions()) -- full-screen shadowmap

local geo = bu.newOcclusionMesh()
--add a bunch of edges
geo:addEdge(w/2-50,h/2-50, w/2+50,h/2-50) -- ax,ay, bx,by
geo:addEdge(w/2+50,h/2-50, w/2+50,h/2+50)
geo:addEdge(w/2+50,h/2+50, w/2-50,h/2+50)
geo:addEdge(w/2-50,h/2+50, w/2-50,h/2-50)

--another new meshes
local circles = bu.newOcclusionMesh()
local a = 0
for i=1, 16 do
	local x,y = math.cos(a), math.sin(a)
		circles:addCircle(w/2+x*180,h/2+y*180,20,24)
	a = a + math.pi/8
end

local lights = bu.newLightsArray()
--add a bunch of lights, all 128 to be precise
for y=1,8 do
	for x=1,16 do
		lights:push( w/16*x-w/32, h*y/8-h/16, 400, .02,.02,.02)
	end
end

local count = geo.edge_count --remember the static count
function love.update(dt)
	--reset edge count to static
	geo.edge_count = count
	--move dynamic meshes
	circles:applyTransform(w/2,h/2,dt/10,1,1,w/2,h/2)
	--and add them to the geometry mesh
	geo:addOcclusionMesh(circles)
	--move one light to the mouse position(also make it brighter)
	local mx,my = love.mouse.getPosition()
	lights:set(1,mx,my,500, .3,.3,.3)
end

function love.draw()
	sm:render(geo, lights)
	lights:draw(sm)
end
```

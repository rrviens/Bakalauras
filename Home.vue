<template>
    <div class="home">
        <div class="legend">
            <div class="legend_icons" style="padding-left: 1%; padding-bottom: 1%;">
                <button style="padding: 10px 10px;" id='block-id-0'></button><b style="padding-left: 5px;">Unvisited node </b>
                <button style="padding: 10px 10px;" id='block-id-4'></button><b> Visited node </b>
                <button style="padding: 10px 10px;" id='block-id-2'></button><b> Start node </b>
                <button style="padding: 10px 10px;" id='block-id-3'></button><b> End node </b>
                <button style="padding: 10px 10px;" id='block-id-1'></button><b> Wall node </b>
                <button style="padding: 10px 10px;" id='block-id-5'></button><b> Shortest path node </b>
            </div>
        </div>
        <span v-html="displaygridData" :key="htmlgridupdate"></span>
        <div style="padding-left: 2%;">
            <div class="update_values" style="display: inline-block;">
                <br />
                <b style="font-size: 30px;"> Update values: </b>
                <br />
                <div>
                    <div><b>Start coordinates:</b> {{sxy}}</div>
                    <input v-model="startx" placeholder="start x" type="number" class="button_width">
                    <input v-model="starty" placeholder="start y" type="number" class="button_width">
                    <span style="padding-left: 10px;"><button v-on:click="updates(startx,starty)" variant="success">Update Start</button></span>
                </div>
                <div>
                    <div><b>End coordinates:</b> {{exy}}</div>
                    <input v-model="endx" placeholder="end x" type="number" class="button_width">
                    <input v-model="endy" placeholder="end y" type="number" class="button_width">
                    <span style="padding-left: 10px;"><button v-on:click="updatee(endx,endy)">Update End</button></span>
                </div>
                <div>
                    <div><b>Set custom wall coordinates:</b></div>
                    <input v-model="wallx" placeholder="wall x" type="number" class="button_width">
                    <input v-model="wally" placeholder="wall y" type="number" class="button_width">
                    <span style="padding-left: 10px;"><button v-on:click="setwall(wallx,wally)" variant="success">Set Wall</button></span>
                </div>
                <div style="padding-top: 10px;">
                    <button v-on:click="randomizeStartEnd(gridSize,gridSize)">Randomize <b>Start - End</b> positions</button>
                </div>
            </div>
            <div class="algo_manip" style="display: inline-block; padding-left: 2%">
                <div>
                    <br />
                    <b style="font-size: 30px;"> Choose an algorithm: </b>
                    <br />
                    <button v-on:click="Dijkstra(sxy,exy)">Start Dijkstra</button>
                    <span style="padding-left: 10px;">
                        <button v-on:click="astar(grid[sxy[1]][sxy[0]],grid[exy[1]][exy[0]])">Start A*</button>
                    </span>
                </div>
                <div>
                    <b style="font-size: 30px;">Grid size:</b>
                    <br />
                    <button v-on:click="grid=[];makeGrid(50, 50)">Grid size 50</button>
                    <span style="padding-left: 10px;">
                        <button v-on:click="grid=[];makeGrid(100, 100)">Grid size 100</button>
                    </span>
                    <span style="padding-left: 10px;">
                        <button v-on:click="grid=[];makeGrid(200, 200)">Grid size 200</button>
                    </span>
                </div>
                <b style="font-size: 30px;">Wall preset:</b>
                <br />
                <button v-on:click="generateVerticalWalls(gridSize, gridSize)">Generate vertical walls</button>
                <span style="padding-left: 10px;">
                    <button v-on:click="generateHorizontalWalls(gridSize, gridSize)">Generate horizontal walls</button>
                </span>
                <span style="padding-left: 10px;">
                    <button v-on:click="generateRandomWalls(gridSize, gridSize)">Random walls</button>
                </span>
            </div>
            <div>
                <b style="font-size: 30px;"> Output LOG: </b>
                <br />
                <span style="white-space: pre;">{{output}}</span>
            </div>
        </div>
    </div>
</template>

<script>
    export default ({

            /*
            ##################################################################################################
            ########################################### SETUP ################################################
            ##################################################################################################
            */

        name: 'Home',
        data() {                                                        // global variables initialization
            return {
                visitednodes: Number(0),                                // global to return for logging
                gridSize: Number(0),                                    // used as return value for wall generation                              
                grid: [],                                               // initializing grid array, rows and collumns
                todis: "",                                              // html template generation with js
                output: [],                                             // list of logging values
                toCheck: [],                                            // list of neighboring nodes
                startx: Number(0),                                      // x coordinate value of starting node
                starty: Number(0),                                      // y coordinate value of starting node
                endx: Number(0),                                        // x coordinate value of end node
                endy: Number(0),                                        // y coordinate value of end node
                sxy: [0, 0],                                            // x, y coordinate values for the update array of the starting node
                exy: [0, 0],                                            // x, y coordinate values for the update array of the end node
                wallx: Number(0),                                       // set x value of wall node from front end 
                wally: Number(0),                                       // set y value of wall node from front end
                displaygridData: "",                                    // html string, triggered by htmlgridupdate key             
                htmlgridupdate: Number(0),                              // trigger value for displaygridData
                currNode: null,                                         // a* reference to current node used 
                gridClass: '',                                          // string value for css manipulation of grid size 
            };
        },
        methods: {                                                      
            addToGrid(adding) {                                         // creating grid with default value of 50
                this.grid.push(adding)                      
            },
            setwall(x, y) {                                             // build wall with the values passed from front end, at the x, y coordinates
                this.grid[y][x].id = 1                                  // wall ID is 1, so we path the value 
                this.displayGrid()                                      // build function building html for 
                this.htmlgridupdate += 1                                // update key value and update grid to display
            },
            updates(x, y) {                                             // update start node from front end
                this.grid[this.sxy[1]][this.sxy[0]].id = 0              // clear old value and path it unvisited node status
                this.sxy = [Number(x), Number(y)]                       // create a start node x, y value array
                this.grid[Number(y)][Number(x)].id = 2                  // set current node id to 2                
                this.displayGrid()                  
                this.htmlgridupdate += 1    
            },
            updatee(x, y) {                                             // update end node from front end
                this.grid[this.exy[1]][this.exy[0]].id = 0              // clear old value and path it unvisited node status
                this.exy = [Number(x), Number(y)]                                                       // create an end node x, y value array
                this.grid[Number(y)][Number(x)].id = 3                  // set current node id to 3 
                this.displayGrid()
                this.htmlgridupdate += 1
            },
            makeGrid(width, height) {                                   // ID LIST: 0 - unchecked; 1 - Wall; 2 - Start; 3 - End; 4 - Visited Cell; 5 - Shortest path
                function Node(x, y, id) {                               // node constructor, takes x, y coordinates and initial ID value 
                    this.x = x                                          // initialization of node specific values
                    this.y = y
                    this.id = id
                    this.visited = false
                    this.prevexists = false
                    this.prev = []
                    this.foundnodes = false
                    this.f = 0                                          // f = calculated total sum of the heuristic (H) and current step cost (G)
                    this.g = 0                                          // g = step cost total number of steps required to walk here from the start tile
                    this.h = 0                                          // h = heuristic estimate heuristic uses manhattan distance. Measures distance from current tile to the goal
                    this.closed = false
                    this.parent = null
                }
                if (width == 50) {                                      // set of CSS values according to the size of the grid and return gridSize variable
                    this.gridClass = 'paddingSmall'
                    this.gridSize = width
                }
                else if (width == 100) {
                    this.gridClass = 'paddingMedium'
                    this.gridSize = width
                }
                else { 
                    this.gridClass = 'paddingBig'
                    this.gridSize = width
                }



                if (this.grid.length === 0) {                           // stops grid being made multiple times
                    for (let y = 0; y < height; y++) {                  // build row for every collumn we create
                        let row = []
                        for (let x = 0; x < width; x++) {
                            var newNode = new Node(x, y, 0)             // for each value in a row we create a node, using the Node constructor
                            row.push(newNode)                           // we put the node value in the the row array
                        }
                        this.addToGrid(row)                             // add the new row array to the grid
                    }
                }
                this.grid[10][10].id = 2                                // default values for start, end points
                this.grid[width-10][height-10].id = 3
                this.exy = [width-10,height-10]
                this.sxy = [10, 10]
                this.displayGrid(width, height)
            },

            /*
            ##################################################################################################
            ########################################### BUTTON FUNCTIONS #####################################
            ##################################################################################################
            */


            randomizeStartEnd(width, height) {                          // randomizer function sets start and end point of the algorithm to random positions, updates grid with those values.
                let s_w_random = Math.floor(Math.random() * width)  
                let s_h_random = Math.floor(Math.random() * height) 

                let e_w_random = Math.floor(Math.random() * width)
                let e_h_random = Math.floor(Math.random() * height) 

                this.grid[this.sxy[1]][this.sxy[0]].id = 0
                this.grid[this.exy[1]][this.exy[0]].id = 0
                this.exy = [s_w_random, s_h_random]
                this.sxy = [e_w_random, e_h_random]

                this.grid[s_h_random][s_w_random].id = 2
                this.grid[e_h_random][e_w_random].id = 3
                this.displayGrid()
                this.htmlgridupdate += 1
            },

            generateVerticalWalls(width, height) {                          // builds random walls according to how big the grid is, walls are vertical
                if (width <= 50 || height <= 50) {                          // this is very intensive and browsers usually don't handle real time building like this well, so wall building is limited to certain values
                    for (let y = 0; y < height; y++) {
                        for (let x = 0; x < width; x++) {                   // every 2nd width coordinate is has a random, but high chance of having a wall set in its' position. Walls are not set in place of start/end positions
                            if (x % 2 !== 0 && this.grid[y][x].id !== 2 && this.grid[y][x].id !== 3 && Math.random() < 0.9) {
                                this.grid[y][x].id == 1
                                this.changeGrid(x, y)
                            }
                        }
                    }
                }
                else {
                    alert("For performance reasons, wall generation works only on grid sizes 50 and 100.")
                }
            },

            generateHorizontalWalls(width, height) {                        // same as vertical walls
                if (width <= 50 || height <= 50) {
                    for (let y = 0; y < width; y++) {
                        for (let x = 0; x < height; x++) {
                            if (y % 2 !== 0 && this.grid[y][x].id !== 2 && this.grid[y][x].id !== 3 && Math.random() < 0.9) {
                                this.grid[x][y].id == 1
                                this.changeGrid(x, y)
                            }
                        }
                    }
                }
                else {
                    alert("For performance reasons, wall generation works only on grid sizes 50 and 100.")
                }
            },

            generateRandomWalls(width, height) {                                                                                            // chooses whether to use vertical or horizontal scanning randomly
                let decide_random = Math.random()                           
                if (decide_random > 0.5) {
                    for (let y = 0; y < height; y++) {
                        for (let x = 0; x < width; x++) {
                            if (width <= 50) {                                                                                          // performance limitation to not overload the browser 
                                if (x % 2 !== 0 && this.grid[y][x].id !== 2 && this.grid[y][x].id !== 3 && Math.random() < 0.5) {
                                    this.grid[y][x].id == 1
                                    this.changeGrid(x, y)
                                }
                            }
                            else if (width == 100) {
                                if (x % 2 !== 0 && this.grid[y][x].id !== 2 && this.grid[y][x].id !== 3 && Math.random() < 0.1) {
                                    this.grid[y][x].id == 1
                                    this.changeGrid(x, y)
                                }
                            }
                            else {
                                alert("For performance reasons, wall randomizer works only on grid sizes 50 and 100.")
                                return
                            }
                        }
                    }
                }
                else {
                    for (let y = 0; y < width; y++) {
                        for (let x = 0; x < height; x++) {
                            if (width <= 50) {
                                if (y % 2 !== 0 && this.grid[y][x].id !== 2 && this.grid[y][x].id !== 3 && Math.random() < 0.5) {
                                    this.grid[x][y].id == 1
                                    this.changeGrid(x, y)
                                }
                            }
                            else if (width == 100) {
                                if (y % 2 !== 0 && this.grid[y][x].id !== 2 && this.grid[y][x].id !== 3 && Math.random() < 0.1) {
                                    this.grid[x][y].id == 1
                                    this.changeGrid(x, y)
                                 }
                            }
                            else {
                                 alert("For performance reasons, wall randomizer works only on grid sizes 50 and 100.")
                                 return
                                 }
                            }
                        }
                    }
            },

            
            //##################################################################################################
            //########################################### HTML CONSTRUCTOR #####################################
            //##################################################################################################
            


            displayGrid() {                                                                                                                                                             // html constructor to display grid
                this.todis = "<div id='block_container' style='padding-left: 2%; padding-top: 1%;' class = 'block'>"
                for (const row of this.grid) {
                    for (let node of row) {
                        this.todis += "<button v-on:click='setwall("+ node.x + "," + node.y + ")' class=" + this.gridClass + " id='block-id-" + node.id + "'></button>"
                    }
                    this.todis += "</div><div id='block_container' style='padding-left: 2%;'>"
                }
                this.todis += "</div>"
                this.displaygridData = this.todis                           
            },

            changeGrid(thisx, thisy) {                                      // change current grid id to 1
                this.grid[thisy][thisx].id = 1
                this.displayGrid()
                this.htmlgridupdate += 1
            },

            
            //##################################################################################################
            //########################################### DIJKSTRA #############################################
            //##################################################################################################
            

            findNeighbours(x, y) {                                          // dijkstra find neighbours function
                let PosNeighbours = []                                      // positions of new neighbours
                let cnode = null                                            // checked node
                this.grid[y][x].visited = true                              // sets current node we are in to visited
                this.grid[y][x].foundnodes = true                           // sets current node to found
                this.visitednodes += 1                                      // count for logging  
                if (this.grid[y][x].id === 0) {                             // sets unvisited node to visited      
                    this.grid[y][x].id = 4
                }

                cnode = this.grid[y][x + 1]                                 // node to check is 1 to the right of current node
                if (typeof cnode !== "undefined") {                         // if the node is not undefined
                    if (cnode.id !== 1) {                                   // it is not a wall
                        if (!cnode.visited) {                               // is not visited
                            PosNeighbours.push(this.grid[y][x + 1])         // we put the node into neighbours array
                            this.grid[y][x + 1].prev = this.grid[y][x]      // define our previous grid position
                            this.grid[y][x + 1].prevexists = true           // set grid's previous position to true
                        }
                    }
                }
                cnode = this.grid[y][x - 1]                                 // node to check is 1 to the left of current node
                if (typeof cnode !== "undefined") {
                    if (cnode.id !== 1) {
                        if (!cnode.visited) {
                            PosNeighbours.push(this.grid[y][x - 1])
                            this.grid[y][x - 1].prev = this.grid[y][x]
                            this.grid[y][x - 1].prevexists = true
                        }
                    }
                }
                if (typeof this.grid[y + 1] !== "undefined") {              // node to check is 1 node above of current node
                    cnode = this.grid[y + 1][x]
                    if (typeof cnode !== "undefined") {
                        if (cnode.id !== 1) {
                            if (!cnode.visited) {
                                PosNeighbours.push(this.grid[y + 1][x])
                                this.grid[y + 1][x].prev = this.grid[y][x]
                                this.grid[y + 1][x].prevexists = true
                            }
                        }
                    }
                }
                if (typeof this.grid[y - 1] !== "undefined") {              // node to check is 1 node below of current node
                    cnode = this.grid[y - 1][x]
                    if (typeof cnode !== "undefined") {
                        if (cnode.id !== 1) {
                            if (!cnode.visited) {
                                PosNeighbours.push(this.grid[y - 1][x])
                                this.grid[y - 1][x].prev = this.grid[y][x]
                                this.grid[y - 1][x].prevexists = true
                            }
                        }
                    }
                }
                return PosNeighbours.reverse()                              // return neighbours list in reverse
            },

            Dijkstra(start, end) {                                                          // dijkstra algorithm takes start and end positions for the algorithm
                var t1 = performance.now()                                                  // performance measuring 
                this.grid[start[1]][start[0]].id = 2                                        // paint start and end positions with new ids
                this.grid[end[1]][end[0]].id = 3
                if (start !== end) {                                        
                    let startNode = this.grid[start[1]][start[0]]                           // set start node 
                    this.toCheck = []                                                       // nodes to check array
                    this.toCheck.push(startNode)                                            // push startnode into the array
                    let toChecknext = []                                                    // checking next array
                    while (this.grid[end[1]][end[0]].visited === false) {                   // as long as we have not visited the end node, proceed with algorithm
                        if (this.toCheck.length !== 0) {                                    // as long as there are nodes to check
                            for (let node of this.toCheck) {                                // go through nodes
                                if (!node.foundnodes) {                                     // if they are not found yet, we send them to have their neighbours checked
                                    let nodestoadd = this.findNeighbours(node.x, node.y)
                                    for (let nodepush of nodestoadd) {                      // add neighbouring cells to nodes that need to be checked list
                                        toChecknext.push(nodepush)
                                    }
                                }
                            }
                            this.toCheck = []                                               // clear the array   
                        } else {
                            if (toChecknext.length < 1) {                                   // if there is no path, output no path log
                                let t2 = performance.now()
                                let timetaken = (t2 - t1)
                                let amountnodes = 0
                                this.displayGrid()
                                this.output = ""
                                this.output += "Dijkstra: \n"
                                this.output += "Time Taken: " + timetaken.toFixed(2) + " ms \n"
                                this.output += "Nodes In Path: " + amountnodes + "\n"
                                this.output += "Nodes Visited: " + this.visitednodes + "\n"
                                this.output += "THERE IS NO PATH."
                                this.visitednodes = 0
                                return this.output
                            }
                            for (let nodep of toChecknext) {                  // if we run out of nodes to check, we clear next nodes array
                                this.toCheck.push(nodep)
                                }
                                toChecknext = [];
                            
                        }

                    }
                    let currentNode = this.grid[end[1]][end[0]]                // current goal node 
                    let pathnodes = [currentNode]                              // pathnodes converts current goal node to a set of values
                    while (currentNode !== startNode) {
                        pathnodes.push(currentNode.prev)                       // as long as we are not at the end, we need to fill up our previous node pool
                        currentNode = currentNode.prev                         // previous node pool is used to stop scanning for neighboring cells of already scanned nodes
                    }
                    let pathnodexy = []                                        // initializing shortest path array
                    for (let node of pathnodes) {                              // draw shortest path by setting IDs
                        pathnodexy.push([node.x, node.y])
                        if (this.grid[node.y][node.x].id == 4) {
                            this.grid[node.y][node.x].id = 5
                        }
                    }
                    
                    var t2 = performance.now()                                  // performance logging 
                    let timetaken = (t2 - t1)                                   // log output
                    let amountnodes = pathnodexy.length
                    this.displayGrid()
                    this.output = ""
                    this.output += "Dijkstra: \n"
                    this.output += "Time Taken: " + timetaken.toFixed(2) + " ms \n"
                    this.output += "Nodes In Path: " + (amountnodes-1) + "\n"
                    this.output += "Nodes Visited: " + this.visitednodes +"\n"
                    this.visitednodes = 0
                } else {
                    this.output = [start]
                    return [start]
                }
                return this.output
            },

            /*
            ##################################################################################################
            ########################################### A STAR ###############################################
            ##################################################################################################
            */

            manhattan(current, end) {                                           // manhattan heuristic, returns value from end and current values
                var val1 = Math.abs(end.x - current.x)
                var val2 = Math.abs(end.y - current.y)
                return (val1 + val2)
            },
            astarnei(x, y) {                                                    // a* neighbour searching function
                var topath = []                                                 // output initialization array
                this.grid[y][x].visited = true                                  
                this.visitednodes += 1                                          // visited nodes marked as visited        
                if (this.grid[y][x].id === 0) {                                 // as the algorithm passes nodes, marks them visited
                    this.grid[y][x].id = 4
                }
                if (this.grid[y - 1] && this.grid[y - 1][x]) {                  // checks if there are valid nodes around the cell, (not diagonally), returns that node to the neighbor array
                    topath.push(this.grid[y - 1][x]);
                }
                if (this.grid[y + 1] && this.grid[y + 1][x]) {
                    topath.push(this.grid[y + 1][x]);
                }
                if (this.grid[y][x - 1]) {
                    topath.push(this.grid[y][x - 1]);
                }
                if (this.grid[y][x + 1]) {
                    topath.push(this.grid[y][x + 1]);
                }
                return topath                                              // neighbor array
            },
            astar(start, end) {                                           // start of a* algorithm, takes start node and end node, reference of grid uses this.
                                                                                   // logging statistic
                var toCheckList = []                                               // check list
                toCheckList.push(start)                                            // check list filled with start node
            
                var t1 = performance.now()
                while (toCheckList.length > 0) {
                    var lowestIdx = 0                                             // as long as we have something in the check list
                    this.visitednodes += 1                                              
                    for (let i = 0; i < toCheckList.length; i++) {                 // go through check list, compare f values, set lowest index value to current value if it has lower f value
                        if (toCheckList[i].f < toCheckList[lowestIdx].f) { 
                            lowestIdx = i
                        }
                    }
                    var currNode = toCheckList[lowestIdx]                               // currently checked node 
                    if (currNode == end) {                                      // if the currently checked node is the end node
                        let pathNode = currNode                                 // go through parent nodes of path, create path array
                        let path = []
                        while (pathNode.parent) {                               // as long as there is a parent node of the currently traversed node, we put that node into path array
                            path.push(pathNode)
                            pathNode = pathNode.parent                          // set currently traversed node to the parent node, go again
                        }
                        for (var node of path) {                                // as long as the node id's are not 2/3 (start/finish), we paint them as path nodes
                            if (this.grid[node.y][node.x].id !== 2 && this.grid[node.y][node.x].id !== 3 ) {
                                this.grid[node.y][node.x].id = 5
                            }
                        }
                        var t2 = performance.now()                              // logging output
                        let timetaken = (t2 - t1)
                        this.displayGrid()
                        this.output = ""
                        this.output += "A STAR: \n"
                        this.output += "Time taken: " + timetaken.toFixed(2) + " ms \n"
                        this.output += "Nodes In Path: " + path.length + "\n"
                        this.output += "Nodes Visited: " + Math.floor(this.visitednodes / 2) + "\n"
                        this.visitednodes = 0
                        return
                    }
                    toCheckList.splice(lowestIdx, 1)                            // replace first element of array w/ lowest index value
                    currNode.closed = true;                                     // set node to closed
                    var neighbors = this.astarnei(currNode.x, currNode.y)
                    for (let i = 0; i < neighbors.length; i++) {
                        var neighbor = neighbors[i]
                        if (neighbor.closed || neighbor.id === 1) { 
                            continue
                        }
                        var g_sc = currNode.g + 1
                        var g_scIsBest = false
                        if (!neighbor.visited) { 
                            g_scIsBest = true
                            neighbor.h = this.manhattan(neighbor, end) 
                            neighbor.visited = true 
                            toCheckList.push(neighbor)
                        } else if (g_sc < neighbor.g) {
                            g_scIsBest = true
                        }
                        if (g_scIsBest) {
                            neighbor.parent = currNode  
                            neighbor.g = g_sc 
                            neighbor.f = neighbor.g + neighbor.h 
                        }
                    }
                }
                this.displayGrid()
                return this.output
            }
        },

        created: function () {                                                  // default grid size on startup
            this.makeGrid(50, 50)
            this.displayGrid()
        }
    });
</script>

<style>
    span button{
        pointer-events: auto;
    }
    .button_width{
        width: 50px;
    }
    .legend_icons{
        height: 16px;
    }
    .legend{
        padding-left: 1%;
    }
    .paddingBig {
        padding: 2px 2px
    }
    .paddingMedium{
        padding: 4px 4px
    }
    .paddingSmall{
        padding: 6px 6px
    }

    #block_container {
        overflow: auto;
        max-height: 80%;
        max-width: 80%;
        width: 80%;
        height: 80%;
        text-align: left;
        padding: 0px;
        font-size: 0;
        pointer-events: auto;
    }

    /* empty cell */
    #block-id-0 {
        display: inline-block;
        background-color: #e1e0e0;
        border: solid;
        border-width: 1px;
        text-align: center;
        text-decoration: none;
        font-size: 0px;
        pointer-events: auto;
    }
    /* wall */
    #block-id-1 {
        display: inline-block;
        background-color: #0b6066;
        border: solid;
        border-width: 1px;
        text-align: center;
        text-decoration: none;
        font-size: 0px;
        pointer-events: auto;
    }
    /* start node */
    #block-id-2 {
        display: inline-block;
        background-color: #64c190;
        border: solid;
        border-width: 1px;
        text-align: center;
        text-decoration: none;
        font-size: 0px;
        pointer-events: auto;
    }
    /* end node */
    #block-id-3 {
        display: inline-block;
        background-color: #ff4554;
        border: solid;
        border-width: 1px;
        text-align: center;
        text-decoration: none;
        font-size: 0px;
        pointer-events: auto;
    }
    /* visited node */
    #block-id-4 {
        display: inline-block;
        background-color: #828282;
        border: solid;
        border-width: 1px;
        text-align: center;
        text-decoration: none;
        font-size: 0px;
        pointer-events: auto;
    }
    /* shortest path */
    #block-id-5 {
        display: inline-block;
        background-color: #9babfb;
        border: solid;
        border-width: 1px;
        text-align: center;
        text-decoration: none;
        font-size: 0px;
        pointer-events: auto;
    }
</style>




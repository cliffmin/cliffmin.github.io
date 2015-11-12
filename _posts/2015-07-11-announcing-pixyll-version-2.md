<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.6/d3.js"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
<script>
    function makeGraphNodesCycle(size) {
        var resultJson = {
            nodes: [],
            links: []
        };

        var n, i, j, x, y;
        var matrix = [];

        for (n = 0; n < size; n++) {
            resultJson.nodes.push({
                name: n + 1,
                group: n
            });

            matrix.push([]);
            for (x = 0; x < size; x++) {
                matrix[n].push(0);
            }
        }

        matrix[0][1] = 1;
        matrix[0][size - 1] = 1;
        matrix[size - 1][0] = 1;
        matrix[size - 1][size - 2] = 1;

        for (y = 1; y < size - 1; y++) {
            matrix[y - 1][y + 1] = 1;
        }

        for (i = 0; i < matrix.length; i++) {
            for (j = 0; j < matrix.length; j++) {
                if (matrix[i][j] === 1) {
                    resultJson.links.push({
                        source: i,
                        target: j,
                        value: 3
                    })
                }
            }
        }
        return resultJson;
    }

    function getDropDownList(optionList) {
        var combo = $("<select class='subcycles'></select>");

        $.each(optionList, function(i, el) {
            combo.append("<option>" + el + "</option>");
        });

        $("#form #cycles").append(combo);
    }

    /** 
     * Cn is an array of K-graphs [2,2,2] = k2,k2,k2
     * The cycle size is the number of elements in Cn
     *
     * Note: comments referencing "original cycle" mean the 
     * non-converted cycle of size N "old" vertices.
     * The "transformed" cycle is the cycle that has each
     * "new" vertex transformed to K graphs
     */
    function getMatrix(Cn) {
        var matrix = [];

        // count the number of vertices 
        // (dont' just do cycle size * k size, we might have different k sizes at some point)
        var totalCount = 0;
        for (var i = 0; i < Cn.length; i++)
        for (var j = 0; j < Cn[i]; j++)
        totalCount++;

        // we need to calculate a "new index" for each vertex in the original cycle
        var newIndex = 0;

        // we know there are totalCount number of new vertices
        for (var i = 0; i < totalCount; i++) {

            // Cn[i] is the K graph size at this "old" vertex
            for (var j = 0; j < Cn[i]; j++) {

                // initialize the matrix row
                matrix[newIndex] = [];

                // initialize all adjacencies to 0
                for (var k = 0; k < totalCount; k++)
                matrix[newIndex][k] = 0;

                // calculate the shift from [newIndex][0] for this grouping of adjacencies
                var shift = (i - 1) * Cn[(i - 1 + Cn.length) % Cn.length];
                shift = (shift + totalCount) % totalCount;

                // put the shifted group into the matrix
                for (var k = shift; k < shift + 3 * Cn[i]; k++) {
                    var subidx = k % totalCount;
                    if (subidx != newIndex) {
                        matrix[newIndex][subidx] = 1;
                    }
                }

                // increment the next new index
                newIndex++;
            }
        }
        return matrix;
    }

    function splitMatrix(matrix, ksize) {
        var splits = [];
        for (var i = 1; i <= ksize / 2; i++) {

            // create a new copy of matrix as new split
            var split = [];
            for (var j = 0; j < matrix.length; j++) {
                split[j] = [];
                for (var k = 0; k < matrix[j].length; k++) {
                    split[j][k] = matrix[j][k];
                }
            }

            // zero out the cross diagonal at this i size
            for (var x = 0; x < matrix.length; x++) {
                split[x][(x + i) % split[x].length] = 0;
                split[x][((x - i) + split[x].length) % split[x].length] = 0;
            }

            // track split
            splits.push(split);
        }
        return splits;
    }

    function clear() {

        d3.select("svg").remove();

    }

    function makeGraphNodes(matrix, matrixInput) {
        var resultJson = {
            nodes: [],
            links: []
        };
        var n, i, j, name, y, counter = 0,
            linkValue;

        for (n = 0; n < matrixInput.length; n++) {
            for (y = 0; y < matrixInput[n]; y++) {
                counter++;
                resultJson.nodes.push({
                    name: counter,
                    group: n + 1
                });
            }
        }

        for (i = 0; i < matrix.length; i++) {
            for (j = 0; j < matrix.length; j++) {
                if (matrix[i][j] === 1) {
                    if (resultJson.nodes[i].group === resultJson.nodes[j].group) {
                        linkValue = 20;
                    } else linkValue = 4;

                    resultJson.links.push({
                        source: i,
                        target: j,
                        value: linkValue
                    })

                }
            }
        }

        return resultJson;
        //{
        //"nodes":[
        //{"name":"node0","group":1},
        //"links":[
        //{"source":"node0","target":"node1","value":3}
        //] };  
    }
    
    $('#cycleSize').change(function () {
    var size = parseInt($("#cycleSize").val());
    $(".subcycles").remove();

    $("#form #cycles").html("<p class='subcycles'>Select options: </p>");
    for (var i = 0; i < size; i++) {
        getDropDownList([2, 3, 4, 5]);
    }
    $("#form #cycles").append('<input id="bundle" class="subcycles" type="checkbox">Bundle<br>');
});

$("#generate").click(function () {
    clear();

    // get all the inputs into an array.
    var $inputs = $('#form :input');

    var matrixInput = [];
    $inputs.each(function () {
        matrixInput.push(parseInt($(this).val()));
    });
    var cycleSize = matrixInput.shift();
    var bundleBool = matrixInput.pop();
    bundleBool = $('#bundle').is(':checked');

    (function (cycleSize, bundleBool, matrixInput) {
        var width = 960,
            height = 500;
        var graph = makeGraphNodes(getMatrix(matrixInput), matrixInput);
        var color = d3.scale.category20();

        var force = d3.layout.force()
            .charge(-120)
            .linkDistance(function(d) {
                if(bundleBool)
                return  width/d.value;
                else return width/4;
            }) 
            .size([width, height]);

        var svg = d3.select("body").append("svg")
            .attr("width", width)
            .attr("height", height);

        var drawGraph = function (graph) {
            force.nodes(graph.nodes)
                .links(graph.links)
                .start();

            var link = svg.selectAll(".link")
                .data(graph.links)
                .enter().append("line")
                .attr("class", "link")
                .style("stroke-width", function (d) {
                return Math.sqrt(d.value);
            });

            var gnodes = svg.selectAll('g.gnode')
                .data(graph.nodes)
                .enter()
                .append('g')
                .classed('gnode', true);

            var node = gnodes.append("circle")
                .attr("class", "node")
                .attr("r", 5)
                .style("fill", function (d) {
                return color(d.group);
            })
                .call(force.drag);

            var labels = gnodes.append("text")
                .attr("dx", 12)
                .attr("dy", ".35em")
                .text(function (d) {
                return d.name;
            });

            force.on("tick", function () {
                link.attr("x1", function (d) {
                    return d.source.x;
                })
                    .attr("y1", function (d) {
                    return d.source.y;
                })
                    .attr("x2", function (d) {
                    return d.target.x;
                })
                    .attr("y2", function (d) {
                    return d.target.y;
                });

                gnodes.attr("transform", function (d) {
                    return 'translate(' + [d.x, d.y] + ')';
                });
            });
        };


        drawGraph(graph);
        //drawGraph(makeGraphNodesCycle(cycleSize));

    }(cycleSize, bundleBool, matrixInput))
});
</script>

<form id="form">Choose your Cn:
    <select id='cycleSize'>
        <option value="0">0</option>
        <option value="3">3</option>
        <option value="4">4</option>
        <option value="5">5</option>
        <option value="6">6</option>
        <option value="7">7</option>
    </select>
    <div id="cycles"></div>
</form>
<div id="generate">Generate</div>
</br>
<div id="clear">Clear</div>

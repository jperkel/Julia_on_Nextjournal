# Julia on Nextjournal

This computational notebook demonstrates a few features of the Julia language. It was written to accompany a *Nature *Toolbox article on Julia, published 30 July 2019 (*[Nature](https://www.nature.com/articles/d41586-019-02310-3)*[ ](https://www.nature.com/articles/d41586-019-02310-3)**[572](https://www.nature.com/articles/d41586-019-02310-3)**[, 141-142 (2019)](https://www.nature.com/articles/d41586-019-02310-3)).

## Syntactic sugar

Julia allows programmers to insert Greek characters (really, any Unicode symbol) into their code, making it easier to read. Some of these symbols are predefined, such as pi (3.141593...). We'll use that symbol to calculate the area and circumference of a circle. (Type `("\\pi [TAB]")` to create the pi symbol.)

```julia id=e773115a-fe11-43de-be91-105c5fbec5a9
# We use semicolons to suppress the output of certain cells
r = 5; # radius of a circle
A = π * r^2;
C = 2 * π * r;
```

```julia id=2f4938d4-9b93-4e17-a423-910fa396cd8f
println("Area = ", round(A, digits = 2))
println("Circumference = ", round(C, digits = 2))
```

Greek characters can also represent functions. Here, we'll define two functions f(x) and g(x), and use the Greek sigma $\\Sigma$ (`("\\Sigma")`) to sum those functions over an interval.

```julia id=6e5396f6-4a21-439c-84c6-c2522e24e8d3
function ∑(f::Function, from::Integer, to::Integer)
  # note that Julia allows you to specify the 'type' of parameters passed to 
  # a function, which makes for error-checked and self-documenting code. 
  # Here, the parameter 'f' is defined as a function; 'from' and 'to'
  # are defined as integers.
    sum = 0
    for i = from:to
        sum += f(i)
    end
    return sum
end
```

```julia id=2b2744d0-201f-49d8-98bb-b2621a1add47
f(x) = 2x + 4 # function `f` (note that no multiplication symbol is required between 2 and x.)
g(x) = x^2 + 4 # function `g`

sum = ∑(f,1,10);
# note the '$sum' string formatting syntax
println("The sum of \'f(x) = 2x + 4\' over the interval [1,10] is: $sum") 
sum = ∑(g,1,10);
println("The sum of \'g(x) = x^2 + 4\' over the interval [1,10] is: $sum")
```

If you're so inclined, you can even use emoji as variable and function names. How about a 'smiley cat'? (Type `("\\:smiley_cat: [TAB]")`)

```julia id=84f97633-cc14-4cf7-af8e-255d97c11770
😺= 7
for i in 1:😺
  println(i)
end
```

## Distributed computing

Julia also supports distributed computing -- the distribution of computational work across multiple processor cores, whether locally or remotely. The following example was generously provided by [Simon Byrne](https://clima.caltech.edu/our-team/), a software engineer at Caltech and member of the Climate Modeling Alliance team.

Consider a simple Monte Carlo estimation of $\\pi$ (see [here](http://mathfaculty.fullerton.edu/mathews/n2003/montecarlopimod.html) for further details). If we generate uniform random variables $x, y \\in \[0,1\]$, then

$$
P(|x^2 + y^2| <= 1) = \frac{\pi}{4}
$$

Then we can estimate $\\pi$ by sampling $x,y$ and scaling the proportion by 4.

Firstly we consider a simple serial implementation, where we accumulate into a variable `("c")`:

```julia id=9f7d4405-d199-423a-9a3d-694d5872be77
function mc_pi(n)
    c = 0
    for i = 1:n
        x = rand()
        y = rand()
        c += (x^2 + y^2 <= 1)
    end
    c/n*4
end
```

Julia code is compiled 'just-in-time'. The first time it runs, it is relatively slow. So, we run it twice, and time the output. We run the calculation a billion times.

```julia id=4c4336b8-e273-46f1-854c-9fc89a7d94cd
@time mc_pi(10^9)
```

This second run takes advantage of precompiled code, and thus should be slightly faster

```julia id=050feae2-be49-49ab-a38b-ee696cadd50c
@time mc_pi(10^9)
```

To do this in parallel, first we load the `("Distributed")` package (which comes with Julia), and add some processes

```julia id=53020841-6550-4509-b791-f39fbaea7037
using Distributed
localprocs = addprocs(4); # start 4 workers on my local computer
```

We need to slightly rewrite the code to be amenable to parallelism: in this case, we use the [`("@distributed")`](https://docs.julialang.org/en/v1/stdlib/Distributed/index.html#Distributed.@distributed) macro, specifying a "reducer" function (in this case, `("+")`), which will combine the outputs of all the loops.

```julia id=f0c00852-54a4-4f15-a8b8-ca0384331ca8
function mc_pi_parallel(n)
    c = @distributed (+) for i = 1:n
        x = rand()
        y = rand()
        x^2 + y^2 <= 1
    end
    c/n*4
end
```

Again, the first time we run this, the code is slowed due to JIT compilation overhead

```julia id=119ceff0-bbe1-44d0-93bf-f367cc243998
@time mc_pi_parallel(10^9)
```

If we rerun the code, we should see a small speedup -- my local machine only has 2 physical cores. Still, the code is about 43% faster than the non-parallelized implementation above (2.4 sec vs 4.2 sec). And it's also faster than the first run of `("mc_pi_parallel()")`, which had to be compiled.

```julia id=e07ef2fa-3738-4096-a387-5718f543237e
@time mc_pi_parallel(10^9) # only a small speedup
```

```julia id=33054720-726d-4c77-8701-3d807037b1ac
rmprocs(localprocs...) # release the allocated workers...
```

## Graphing

Next, we'll graph the Fibonacci sequence, where each value equals the sum of the two previous values: $F\_n = F\_{n-1} + F\_{n-2}$ Thus, the sequence is: 0, 1, 1, 2, 3, 5, 8, ...

```julia id=2d963984-e6f4-48ba-ad2c-be658850f4af
function fibonacci(i = 25) 
    # compute the Fibonacci sequence. By default, return the first 25 values.
    @assert(i >= 2) # basic input check
    f1 = 0
    f2 = 1
    result = [f1, f2]
    for i in 1:(i-2)
        f3 = f1 + f2
        push!(result, f3) # add the new value to the array
        f1 = f2
        f2 = f3
    end
    return result
end
```

```julia id=19f3ffbf-4d22-40b2-876b-6b6ee943056a
myarray = fibonacci();
```

We use Julia's package manager to load the libraries needed to plot our results.

```julia id=197e4109-e259-4105-b4c6-c61c3479c475
using Pkg
```

```julia id=4aeb1ad7-2296-4043-84df-942bf189f3ba
Pkg.add("Plots")
using Plots
Pkg.add("PyPlot")
pyplot()
```

```julia id=01966056-cedd-4c43-a093-94cb0a053367
plot(1:25,myarray,label="",color="blue", markershape = :circle, markercolor = :red, 
    xlabel = "Sequence no.", ylabel = "Fibonacci no.", title = "The first 25 Fibonacci numbers")
```

## Sequence analysis

Julia can also handle DNA sequence analysis, using the BioJulia packages.

```julia id=52825213-0fc8-443c-85ac-600b3a9985d6
Pkg.add("BioSequences")
using BioSequences
```

First, we'll read in a sequence file -- a circular DNA, called a plasmid, from the bacterium, *Yersinia pestis*. `("sequence.fasta")` is a FASTA-formatted file downloaded from [https://www.ncbi.nlm.nih.gov/nuccore/NC\_005816.1?report=fasta&log\$=seqview](https://www.ncbi.nlm.nih.gov/nuccore/NC_005816.1?report=fasta&log$=seqview).

[sequence.fasta][nextjournal#file#58431a11-e0de-4229-981f-5f921211fe8c]

```julia id=60a8c347-795d-46bb-8ec9-764428a71d83
reader = FASTA.Reader(open($$ref{{["~:output","58431a11-e0de-4229-981f-5f921211fe8c",null]}},"r"))
my_dna = read(reader)
close(reader)
```

Convert the file into a BioJulia `("DNASequence")` object.

```julia id=634331aa-c889-4e14-9d63-6cc9902b711b
my_dna = DNASequence(sequence(my_dna))
```

From [https://github.com/jperkel/example\_notebook/blob/master/My\_sample\_notebook.ipynb](https://github.com/jperkel/example_notebook/blob/master/My_sample_notebook.ipynb), we know that there is a 'hypothetical protein' located at 3485:3857 in this plasmid. But those coordinates are from Python, which is zero-indexed (that is, the first element of an array is 0, not 1). For Julia, which is not (the first element is 1), the location would be 3486:3857. Here, we extract that gene sequence, convert it to RNA, and translate it.

```julia id=4c9d5dea-f72d-44b8-aca4-41ea925e4e01
my_gene=my_dna[3486:3857]
my_rna=RNASequence(my_gene)
```

```julia id=b6983a8e-71ec-4670-bade-fcb0c196316e
my_ptn = BioSequences.translate(my_rna)
```

Now let's see what info the NCBI Entrez database has on this protein. From [https://github.com/jperkel/example\_notebook/blob/master/My\_sample\_notebook.ipynb ](https://github.com/jperkel/example_notebook/blob/master/My_sample_notebook.ipynb )we learned that the accession number for this sequence is NP\_995570.1.

```julia id=a8572bd2-5885-43e3-964f-d9e8fd203967
Pkg.add("BioServices")
using BioServices.EUtils
Pkg.add("EzXML")
using EzXML
```

```julia id=e603bfa9-2dda-4d6a-a917-6b1792c9084c
res = efetch(db="protein",id="NP_995570.1",rettype="gb",retmode="xml");
```

We'll need to parse the XML output. What gene is this?

```julia id=1df041a5-e5b1-4108-9bf5-71e93f99256d
doc = parsexml(res.body)
seq = findfirst("/GBSet/GBSeq",doc)
nodecontent(findfirst("GBSeq_definition", seq))
```

What is its sequence?

```julia id=62f2e4be-f545-47fb-8ba0-9fbefd178eb8
entrez_ptn = AminoAcidSequence(nodecontent(findfirst("GBSeq_sequence",seq)))
```

Let's compare the Entrez sequence with our FASTA sequence using a pairwise alignment...

```julia id=7ca03e61-8c35-4797-9337-ba5b953907cd
Pkg.add("BioAlignments")
using BioAlignments
```

```julia id=a7e17d95-8f7e-4878-b6d1-ab1341f2c286
costmodel = CostModel(match=0, mismatch=1, insertion=1, deletion=1);
pairalign(EditDistance(), my_ptn, entrez_ptn, costmodel)
```

It's a match!

## Calling Python

Finally, we'll demonstrate Julia's ability to utilize code written in another language, using the `("PyCall")` package to call the Python `("folium")` mapping library (see our June 2018 [mapping feature](https://www.nature.com/articles/d41586-018-05331-6)).

```julia id=2808146f-2089-4b98-9d65-9736835a6494
Pkg.add("PyCall")
using PyCall
```

If you don't have `("folium")` installed, you'll need to do so. You can do that here using the Julia `("Conda")` package. 

```julia id=612e92ea-cb46-43be-a125-dcce8fb6b993
Pkg.add("Conda")
using Conda
```

```julia id=d45d500c-31c1-4ea0-bb42-c5f252065a66
Conda.add_channel("conda-forge")
Conda.add("folium")
```

Now we create a simple map: a few points in London, Oxford and Cambridge, overlaid on either a street map, or on a map of geological data provided by the [Macrostrat Project](https://macrostrat.org/). Note that the code below is written in Python. By bracketing the code with the `("py")` macro (`("py\"\"\"{code}\"\"\"")` or `("py\"{code}\"")`), Julia knows to pass that code to `("PyCall")` directly.

The resulting map is interactive: you can zoom, pan, click the points of interest, and alternate between the two layers (by clicking on the tiles icon in the upper-right corner of the map). If you click anywhere on the map, a popup will appear showing the latitude and longitude of that position. \[NOTE: IF THE MAP RENDERS WITH NO LAYER SHOWING, CLICK THE TILES ICON IN THE UPPER-RIGHT CORNER TO ACTIVATE THEM\]

```julia id=0da3564c-0703-4a1c-ab91-1f95e10754db
py"""
import folium

def draw_map():
    coords = { 
        0: { "name": "Nature", "lat": 51.533925, "long": -0.121553 },
        1: { "name": "Francis Crick Institite", "lat": 51.531877, "long": -0.128767 },
        2: { "name": "MRC Laboratory for Molecular Cell Biology", "lat": 51.524435, "long": -0.132495 },
        3: { "name": "Kings College London", "lat": 51.511573, "long": -0.116083 },
        4: { "name": "Imperial College London", "lat": 51.498780, "long": -0.174888 },
        5: { "name": "Cambridge University", "lat": 52.206960, "long": 0.115034 },
        6: { "name": "Oxford University", "lat": 51.754843, "long": -1.254302 },
        7: { "name": "Platform 9 3/4", "lat": 51.532349, "long": -0.123806 }
    }
    
    m = folium.Map(location = [51.8561, -0.2966], tiles = 'CartoDB positron', zoom_start = 9)
		
    # add the points of interest...
    for key in coords.keys():
        folium.CircleMarker(
        location=[coords[key]['lat'], coords[key]['long']],
        popup=coords[key]['name'],
        color=('crimson' if coords[key]['name'] == 'Nature' else 'blue'),
        fill=False,
        ).add_to(m)
    
    # pull in the Macrostrat tile layer
    folium.TileLayer(tiles='https://tiles.macrostrat.org/carto/{z}/{x}/{y}.png', 
                 attr='Macrostrat', name='Macrostrat').add_to(m)
    folium.LayerControl().add_to(m) # allow user to switch between map layers
    folium.LatLngPopup().add_to(m) # provide a lat/long popup

    return m
"""

# draw the map
py"draw_map()"
```

## Computational reproducibility

Document session for [computational reproducibility](https://www.nature.com/articles/d41586-018-05990-5).

```julia id=4437e104-0451-419f-b897-85488b2dce3c
versioninfo()
```

```julia id=35901c72-7cdb-4401-9b7c-ef0e9ade86e4
Pkg.installed()
```

[nextjournal#file#58431a11-e0de-4229-981f-5f921211fe8c]: https://nextjournal.com/data/QmeV17dEvDfpddou982RebTu6MUkaPpTmPgnt3cs33Q6SW?content-type=&filename=sequence.fasta

<details id="com.nextjournal.article">
<summary>This notebook was exported from <a href="https://nextjournal.com">https://nextjournal.com</a></summary>

```edn nextjournal-metadata
{:nodes
 {"01966056-cedd-4c43-a093-94cb0a053367"
  {:compute-ref #uuid "41af4312-a7d5-4c0a-87c4-6192ad530834",
   :exec-duration 12471,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "050feae2-be49-49ab-a38b-ee696cadd50c"
  {:compute-ref #uuid "ec45792d-4115-4dfa-aa97-8c2b1543bd5a",
   :exec-duration 4681,
   :kind "code",
   :output-log-lines {:stdout 1},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "0da3564c-0703-4a1c-ab91-1f95e10754db"
  {:compute-ref #uuid "8bd7244b-3baf-4392-8b5f-86efb2f14560",
   :exec-duration 581,
   :kind "code",
   :output-log-lines {},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "119ceff0-bbe1-44d0-93bf-f367cc243998"
  {:compute-ref #uuid "7530f62e-0a9e-473f-a1fd-9effb4f0f6e3",
   :exec-duration 4446,
   :kind "code",
   :output-log-lines {:stdout 1},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "197e4109-e259-4105-b4c6-c61c3479c475"
  {:compute-ref #uuid "892d6d7f-628e-424c-ac2c-30cf66c55319",
   :exec-duration 5,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "19f3ffbf-4d22-40b2-876b-6b6ee943056a"
  {:compute-ref #uuid "5ca4d15e-38db-49eb-9d08-60200040b718",
   :exec-duration 64,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "1df041a5-e5b1-4108-9bf5-71e93f99256d"
  {:compute-ref #uuid "5e5da9f8-352f-4e96-8a13-be8ac27fcae3",
   :exec-duration 8,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "2808146f-2089-4b98-9d65-9736835a6494"
  {:compute-ref #uuid "cb788480-07a8-4a9c-89be-f6cb9272afbc",
   :exec-duration 1216,
   :kind "code",
   :output-log-lines {:stdout 5},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "2b2744d0-201f-49d8-98bb-b2621a1add47"
  {:compute-ref #uuid "14865f82-9092-4aa8-b140-3b62ade23e16",
   :exec-duration 780,
   :kind "code",
   :output-log-lines {:stdout 2},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "2d963984-e6f4-48ba-ad2c-be658850f4af"
  {:compute-ref #uuid "78860133-75cb-4e91-8822-914ab300ce45",
   :exec-duration 660,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "2f4938d4-9b93-4e17-a423-910fa396cd8f"
  {:compute-ref #uuid "61d4c269-3f7b-4a91-8610-6e0702fcece7",
   :exec-duration 557,
   :kind "code",
   :output-log-lines {:stdout 2},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "33054720-726d-4c77-8701-3d807037b1ac"
  {:compute-ref #uuid "2d426360-b7d0-4d7c-ba30-108d6c25cfe1",
   :exec-duration 523,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "35901c72-7cdb-4401-9b7c-ef0e9ade86e4"
  {:compute-ref #uuid "2644a481-45e7-4209-ba80-ecc8e16e4269",
   :exec-duration 20,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "4437e104-0451-419f-b897-85488b2dce3c"
  {:compute-ref #uuid "17dfd3c0-e4d2-4b85-bdbd-bbe29df2618e",
   :exec-duration 602,
   :kind "code",
   :output-log-lines {:stdout 11},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "4aeb1ad7-2296-4043-84df-942bf189f3ba"
  {:compute-ref #uuid "987e4b69-e91f-4551-8039-90a0931a8d69",
   :exec-duration 2618,
   :kind "code",
   :output-log-lines {:stdout 10},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"],
   :stdout-collapsed? false},
  "4c4336b8-e273-46f1-854c-9fc89a7d94cd"
  {:compute-ref #uuid "cce0fa7b-5726-4b6c-b8a8-a18556880519",
   :exec-duration 5017,
   :kind "code",
   :output-log-lines {:stdout 1},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "4c9d5dea-f72d-44b8-aca4-41ea925e4e01"
  {:compute-ref #uuid "ad761eb6-1b60-4770-a31e-a15f4eaaed45",
   :exec-duration 270,
   :kind "code",
   :output-log-lines {},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "52825213-0fc8-443c-85ac-600b3a9985d6"
  {:compute-ref #uuid "45c6cff3-9ac0-4818-9eb5-05c676d6746f",
   :exec-duration 1302,
   :kind "code",
   :output-log-lines {:stdout 5},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "53020841-6550-4509-b791-f39fbaea7037"
  {:compute-ref #uuid "03674a24-59b4-45da-938f-50fbe814bbf5",
   :exec-duration 5106,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "58431a11-e0de-4229-981f-5f921211fe8c" {:kind "file"},
  "60a8c347-795d-46bb-8ec9-764428a71d83"
  {:compute-ref #uuid "212eb861-ea34-4347-a89c-dfe004da3340",
   :exec-duration 895,
   :kind "code",
   :output-log-lines {},
   :refs
   ({:line 1,
     :from 27,
     :to 92,
     :node [:output "58431a11-e0de-4229-981f-5f921211fe8c" nil],
     :kind "ref"}),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "612e92ea-cb46-43be-a125-dcce8fb6b993"
  {:compute-ref #uuid "902025a3-f972-419f-9ddf-0fc350797c7c",
   :exec-duration 1186,
   :kind "code",
   :output-log-lines {:stdout 5},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "62f2e4be-f545-47fb-8ba0-9fbefd178eb8"
  {:compute-ref #uuid "574e6662-8c9c-4866-b18a-80c4c99578b5",
   :exec-duration 8,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "634331aa-c889-4e14-9d63-6cc9902b711b"
  {:compute-ref #uuid "225bed20-006b-4fdf-8e2f-0493d07410a3",
   :exec-duration 434,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "6e5396f6-4a21-439c-84c6-c2522e24e8d3"
  {:compute-ref #uuid "184cfd4a-0d75-40b9-ba26-e16c7ea5c26b",
   :exec-duration 3514,
   :kind "code",
   :output-log-lines {},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "7ca03e61-8c35-4797-9337-ba5b953907cd"
  {:compute-ref #uuid "353e9beb-9081-4178-b4b7-ce66cffac7be",
   :exec-duration 1209,
   :kind "code",
   :output-log-lines {:stdout 5},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "84f97633-cc14-4cf7-af8e-255d97c11770"
  {:compute-ref #uuid "d3a62a41-9487-4c75-b378-bceaaad9641d",
   :exec-duration 609,
   :kind "code",
   :output-log-lines {:stdout 7},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "9f7d4405-d199-423a-9a3d-694d5872be77"
  {:compute-ref #uuid "edd7708b-81b2-44db-b5fb-c75ea2976670",
   :exec-duration 66,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "a7e17d95-8f7e-4878-b6d1-ab1341f2c286"
  {:compute-ref #uuid "4a03feb5-c770-4816-b821-93c480c63f79",
   :exec-duration 11,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "a8572bd2-5885-43e3-964f-d9e8fd203967"
  {:compute-ref #uuid "c09a8f07-f312-48b9-a65b-fae662681b76",
   :exec-duration 2192,
   :kind "code",
   :output-log-lines {:stdout 10},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "b2157366-f074-4a88-afb0-421f394854d3"
  {:environment
   [:environment
    {:article/nextjournal.id
     #uuid "5b460d39-8c57-43a6-8b13-e217642b0146",
     :change/nextjournal.id
     #uuid "5cc9b90b-0bd4-4e38-bb95-834151b2dc86",
     :node/id "39e3f06d-60bf-4003-ae1a-62e835085aef"}],
   :kind "runtime",
   :language "julia",
   :name nil,
   :resources {:machine-type "n1-standard-4"},
   :type :jupyter},
  "b6983a8e-71ec-4670-bade-fcb0c196316e"
  {:compute-ref #uuid "723d0781-133b-4ba3-aade-bf4cb84fa257",
   :exec-duration 8,
   :kind "code",
   :output-log-lines {},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "d45d500c-31c1-4ea0-bb42-c5f252065a66"
  {:compute-ref #uuid "766a842e-85be-4eb5-8562-0330cbf53534",
   :exec-duration 9285,
   :kind "code",
   :output-log-lines {:stdout 10},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "e07ef2fa-3738-4096-a387-5718f543237e"
  {:compute-ref #uuid "c1c698a4-1141-416c-adba-29cd633bdd96",
   :exec-duration 2909,
   :kind "code",
   :output-log-lines {:stdout 1},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "e603bfa9-2dda-4d6a-a917-6b1792c9084c"
  {:compute-ref #uuid "29feaacf-aa7a-4fb1-83b5-0ae72601e46b",
   :exec-duration 3094,
   :kind "code",
   :output-log-lines {},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "e773115a-fe11-43de-be91-105c5fbec5a9"
  {:compute-ref #uuid "fafba92e-f353-40a6-8250-7aec451d82a6",
   :exec-duration 6,
   :kind "code",
   :name nil,
   :output-log-lines {},
   :refs (),
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]},
  "f0c00852-54a4-4f15-a8b8-ca0384331ca8"
  {:compute-ref #uuid "9b8e075f-8df6-478a-9391-9e997071daba",
   :exec-duration 128,
   :kind "code",
   :output-log-lines {},
   :runtime [:runtime "b2157366-f074-4a88-afb0-421f394854d3"]}}}
```
</details>

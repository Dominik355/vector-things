# vector-things

Currently i am working with data pipeline that deals with advertisement candidate pre-selection based on search-queries.
Delaing with compaction/clustering of data for vector search ( faiss, usearch, kmeans,...)

So what Im thinking:
Implement multi layer hnsw graph. Well hnsw is multi layer by its definition.
What i mean is :
1. create clusters(centroids) with inner IVF (inverted file index) storing the vectors.
2. Instead of IVF the centroid if big enough might have another HNSW graph within itself, so it can be basically infinitely chained hnsw graph where
he basic unit is something like 'centroid' and the top level 'centroid' is a single 1. And so each centroid has 'inner: Option<Centroid>'.
Basically 1 way linked list. This way we can parallelize the search.
Qdrant already does segmentation, but it aint a clustering. So segmentation is inner architecture allowing qdrant. tobe distributed/parallel.
And segment are maintained (merged/created/deleted) by the qdrant. 

Currently with qdrant when clustering should be done we need to do that in 2-step way:
1. query
2. decide whether to upsert new centroid or we found good enough threshold

This canbbe done within a single call where user sets : 
ClusterRequest {
    centroid_threshold: f32, is bigger or equal -> add to given centroid, or create new 1
    vector: Vec<f32/i8/...>,
}

So we cna automatically add that vector to inne rgraph of given centroid and again make it recursive


Just an idea.
Basically FAISS does something similar ? as it uses IVF internally, which agian might be good enough. If every centroid contains only like ones or hundreds of vectors,
we might simply do ANN without building a graph out of them ? .

So basically - "recursive hnsw graph based on clustering". Also the top level sclustering might be cnfigurable ? 
No need to use HNSW on all the layers, we might use like siphash for toplevel clustering and then build hnsw per cluster.

If RAM is a problem and we store data on DISK, this clustering might help us narrowthe data good enough so disk IO is not that big of a deal ? 
.. _clustering:

==========
Clustering
==========

`Clustering <http://en.wikipedia.org/wiki/Cluster_analysis>`__ of
unlabeled data can be performed with the module :mod:`sklearn.cluster`.

Each clustering algorithm comes in two variants: a class, that implements
the `fit` method to learn the clusters on train data, and a function,
that, given train data, returns an array of integer labels corresponding
to the different clusters. For the class, the labels over the training
data can be found in the `labels_` attribute.

.. currentmodule:: sklearn.cluster

.. topic:: Input data

    One important thing to note is that the algorithms implemented in
    this module take different kinds of matrix as input.  On one hand,
    :class:`MeanShift` and :class:`KMeans` take data matrices of shape
    [n_samples, n_features]. These can be obtained from the classes in
    the :mod:`sklearn.feature_extraction` module. On the other hand,
    :class:`AffinityPropagation` and :class:`SpectralClustering` take
    similarity matrices of shape [n_samples, n_samples].  These can be
    obtained from the functions in the :mod:`sklearn.metrics.pairwise`
    module. In other words, :class:`MeanShift` and :class:`KMeans` work
    with points in a vector space, whereas :class:`AffinityPropagation`
    and :class:`SpectralClustering` can work with arbitrary objects, as
    long as a similarity measure exists for such objects.

Overview of clustering methods
===============================

.. figure:: ../auto_examples/cluster/images/plot_cluster_comparison_1.png
   :target: ../auto_examples/cluster/plot_cluster_comparison.html
   :align: center
   :scale: 50

   A comparison of the clustering algorithms in scikit-learn


.. list-table::
   :header-rows: 1
   :widths: 14 15 19 25 20

   * - Method name
     - Parameters
     - Scalability
     - Usecase
     - Geometry (metric used)

   * - :ref:`K-Means <k_means>`
     - number of clusters
     - Very large `n_samples`, medium `n_clusters` with
       :ref:`MiniBatch code <mini_batch_kmeans>`
     - General-purpose, even cluster size, flat geometry, not too many clusters 
     - Distances between points 

   * - :ref:`Affinity propagation <affinity_propagation>`
     - damping, sample preference 
     - Not scalable with n_samples
     - Many clusters, uneven cluster size, non-flat geometry
     - Graph distance (e.g. nearest-neighbor graph)

   * - :ref:`Mean-shift <mean_shift>`
     - bandwidth 
     - Not scalable with n_samples
     - Many clusters, uneven cluster size, non-flat geometry
     - Distances between points 

   * - :ref:`Spectral clustering <spectral_clustering>`
     - number of clusters
     - Medium `n_samples`, small `n_clusters`
     - Few clusters, even cluster size, non-flat geometry
     - Graph distance (e.g. nearest-neighbor graph)

   * - :ref:`Hierarchical clustering <hierarchical_clustering>`
     - number of clusters
     - Large `n_samples` and `n_clusters`
     - Many clusters, possibly connectivity constraints
     - Distances between points 

   * - :ref:`DBSCAN <dbscan>`
     - neighborhood size
     - Very large `n_samples`, medium `n_clusters`
     - Non-flat geometry, uneven cluster sizes
     - Distances between nearest points 

   * - :ref:`Gaussian mixtures <mixture>`
     - many
     - Not scalable
     - Flat geometry, good for density estimation
     - Mahalanobis distances to  centers

Non-flat geometry clustering is useful when the clusters have a specific
shape, i.e. a non-flat manifold, and the standard euclidean distance is
not the right metric. This case arises in the two top rows of the figure
above.

Gaussian mixture models, useful for clustering, are described in
:ref:`another chapter of the documentation <mixture>` dedicated to
mixture models. KMeans can be seen as a special case of Gaussian mixture
model with equal covariance per component.

.. _k_means:

K-means
=======

The :class:`KMeans` algorithm clusters data by trying to separate samples
in n groups of equal variance, minimizing a criterion known as the
'inertia' of the groups. This algorithm requires the number of cluster to
be specified. It scales well to large number of samples and has been used
across a large range of application areas in many different fields. It is
also equivalent to the expectation-maximization algorithm when setting the
covariance matrix to be diagonal, equal and small. The K-means algorithm
aims to choose centroids :math:`C` that minimise the within cluster sum of 
squares objective function with a dataset :math:`X` with :math:`n` samples:

.. math:: J(X, C) = \sum_{i=0}^{n}\min_{\mu_j \in C}(||x_j - \mu_i||^2)

K-means is often referred to as Lloyd's algorithm. In basic terms, the
algorithm has three steps. The first step chooses the initial centroids, with
the most basic method being to choose :math:`k` samples from the dataset
:math:`X`. After initialization, k-means consists of looping between the other
two major steps. The first steps assigns each sample to its nearest centroid.
The second step creates new centroids by taking the mean value of all of the
samples assigned to each previous centroid. The difference between the old
and the new centroids is the inertia and the algorithm repeats these last two
steps until this value is less than a threshold. In other words, it repeats
until the centroids do not move significantly.

.. image:: ../auto_examples/cluster/images/plot_kmeans_digits_1.png
   :target: ../auto_examples/cluster/plot_kmeans_digits.html
   :align: right
   :scale: 35

The algorithm can be identified through the concept of `Voronoi diagrams
<https://en.wikipedia.org/wiki/Voronoi_diagram>`_. First the Voronoi diagram
of the points is calculated using the current centroids. Each segment in the
Voronoi diagram becomes a separate cluster. Secondly, the centroids are updated
to the mean of each segment. The algorithm then repeats this until a stopping
criterion is fulfilled. Usually, as in this implementation, the algorithm stops
when the relative decrease in the objective function between iterations is less
than the given tolerance value.

Given enough time, K-means will always converge, however this may be to a local
minimum. This is highly dependent on the the initialisation of the centroids.
As a result, the computation is often done several times, with different
initialisation of the centroids. One method to help address this issue is the
k-means++ initialisation algorithm, which has been implemented in
scikit-learn (use the ``init='kmeans++'`` parameter). This initialises the
centroids to be (generally) distant from each other, leading to provably better
results than random initialisation.

A parameter can be given to allow K-means to be run in parallel, called
`n_jobs`. Giving this parameter a positive value uses that many processors 
(default=1). A value of -1 uses all processors, with -2 using one less, and so 
on. Parallelization generally speeds up computation at the cost of memory (in
this case, multiple copies of centroids need to be stored, one for each job).

.. warning::

    The parallel version of K-Means is broken on OS X when numpy uses the
    Accelerate Framework. This is expected behavior: Accelerate can be called
    after a fork but you need to execv the subprocess with the python binary
    (which multiprocessing does not do under posix).

K-means can be used for vector quantization. This is achieved using the
transform method of a trained model of :class:`KMeans`.

.. topic:: Examples:

 * :ref:`example_cluster_plot_kmeans_digits.py`: Clustering handwritten digits


.. _mini_batch_kmeans:

Mini Batch K-Means
------------------

The :class:`MiniBatchKMeans` is a variant of the :class:`KMeans` algorithm
using mini-batches, random subset of the dataset, to compute the centroids.

Although the :class:`MiniBatchKMeans` converge faster than the KMeans
version, the quality of the results, measured by the inertia, the sum of
the distance of each points to the nearest centroid, is not as good as
the :class:`KMeans` algorithm.

.. figure:: ../auto_examples/cluster/images/plot_mini_batch_kmeans_1.png
   :target: ../auto_examples/cluster/plot_mini_batch_kmeans.html
   :align: center
   :scale: 100


.. topic:: Examples:

 * :ref:`example_cluster_plot_mini_batch_kmeans.py`: Comparison of KMeans and
   MiniBatchKMeans

 * :ref:`example_document_clustering.py`: Document clustering using sparse
   MiniBatchKMeans

 * :ref:`example_cluster_plot_dict_face_patches.py`


.. topic:: References:

 * `"Web Scale K-Means clustering"
   <http://www.eecs.tufts.edu/~dsculley/papers/fastkmeans.pdf>`_
   D. Sculley, *Proceedings of the 19th international conference on World
   wide web* (2010)

.. _affinity_propagation:

Affinity Propagation
====================

:class:`AffinityPropagation` creates clusters by sending messages between
pairs of samples until convergence. A dataset is then described using a small
number of exemplars, which are identified as those most representative of other
samples. The messages sent between pairs represent the suitability for one
sample to be the exemplar of the other, which is updated in response to the
values from other pairs. This updating happens iteratively until convergence,
at which point the final exemplars are chosen, and hence the final clustering
is given.

.. figure:: ../auto_examples/cluster/images/plot_affinity_propagation_1.png
   :target: ../auto_examples/cluster/plot_affinity_propagation.html
   :align: center
   :scale: 50


Affinity Propagation can be interesting as it chooses the number of
clusters based on the data provided. For this purpose, the two important
parameters are the `preference`, which controls how many examplars are
used, and the `damping` factor.

The main drawback of Affinity Propagation is its complexity. The
algorithm has a time complexity of the order :math:`O(N^2 T)`, where `N`
is the number of samples and `T` is the number of iterations until
convergence. Further, the memory complexity is of the order
:math:`O(N^2)` if a dense similarity matrix is used, but reducible if a
sparse similarity matrix is used. This makes Affinity Propagation most
appropriate for small to medium sized datasets.

.. topic:: Examples:

 * :ref:`example_cluster_plot_affinity_propagation.py`: Affinity
   Propagation on a synthetic 2D datasets with 3 classes.

 * :ref:`example_applications_plot_stock_market.py` Affinity Propagation on
   Financial time series to find groups of companies

**Algorithm description:**
The messages sent between points belong to one of two categories. The first is
the responsibility `r(i, k)`, which is the accumulated evidence that sample `k`
should be the exemplar for sample `i`. The second is the availability `a(i, k)`
which is the accumulated evidence that sample `i` should choose sample `k` to
be its exemplar, and considers the values for all other samples that `k` should
be an exemplar. In this way, exemplars are chosen by samples if they are (1)
similar enough to many samples and (2) chosen by many samples to be
representative of themselves.

More formally, the responsibility of a sample `k` to be the exemplar of sample
`i` is given by:

.. math::

    r(i, k) \leftarrow s(i, k) - max [ a(i, \acute{k}) + s(i, \acute{k}) \forall \acute{k} \neq k ]

Where :math:`s(i, k)` is the similarity between samples `i` and `k`. The
availability of sample `k` to be the exemplar of sample `i` is given by:

.. math::

    a(i, k) \leftarrow min [0, r(k, k) + \sum_{\acute{i}~s.t.~\acute{i} \notin \{i, k\}}{r(\acute{i}, k)}]

To begin with, all values for `r` and `a` are set to zero, and the calculation
of each iterates until convergence.

.. _mean_shift:

Mean Shift
==========

:class:`MeanShift` clusters data by estimating *blobs* in a smooth
density of points matrix. This algorithm automatically sets its numbers
of cluster. It will have difficulties scaling to thousands of samples.
The utility function :func:`estimate_bandwidth` can be used to guess
the optimal bandwidth for :class:`MeanShift` from the data.

.. figure:: ../auto_examples/cluster/images/plot_mean_shift_1.png
   :target: ../auto_examples/cluster/plot_mean_shift.html
   :align: center
   :scale: 50


.. topic:: Examples:

 * :ref:`example_cluster_plot_mean_shift.py`: Mean Shift clustering
   on a synthetic 2D datasets with 3 classes.

.. _spectral_clustering:

Spectral clustering
===================

:class:`SpectralClustering` does a low-dimension embedding of the
affinity matrix between samples, followed by a KMeans in the low
dimensional space. It is especially efficient if the affinity matrix is
sparse and the `pyamg <http://code.google.com/p/pyamg/>`_ module is
installed. SpectralClustering requires the number of clusters to be
specified. It works well for a small number of clusters but is not
advised when using many clusters.

For two clusters, it solves a convex relaxation of the `normalised
cuts <http://www.cs.berkeley.edu/~malik/papers/SM-ncut.pdf>`_ problem on
the similarity graph: cutting the graph in two so that the weight of the
edges cut is small compared to the weights in of edges inside each
cluster. This criteria is especially interesting when working on images:
graph vertices are pixels, and edges of the similarity graph are a
function of the gradient of the image.


.. |noisy_img| image:: ../auto_examples/cluster/images/plot_segmentation_toy_1.png
    :target: ../auto_examples/cluster/plot_segmentation_toy.html
    :scale: 50

.. |segmented_img| image:: ../auto_examples/cluster/images/plot_segmentation_toy_2.png
    :target: ../auto_examples/cluster/plot_segmentation_toy.html
    :scale: 50

.. centered:: |noisy_img| |segmented_img|

.. warning:: Transforming distance to well-behaved similarities

    Note that if the values of your similarity matrix are not well
    distributed, e.g. with negative values or with a distance matrix
    rather than a similarity, the spectral problem will be singular and
    the problem not solvable. In which case it is advised to apply a
    transformation to the entries of the matrix. For instance, in the
    case of a signed distance matrix, is common to apply a heat kernel::

        similarity = np.exp(-beta * distance / distance.std())

    See the examples for such an application.

.. topic:: Examples:

 * :ref:`example_cluster_plot_segmentation_toy.py`: Segmenting objects
   from a noisy background using spectral clustering.

 * :ref:`example_cluster_plot_lena_segmentation.py`: Spectral clustering
   to split the image of lena in regions.

.. |lena_kmeans| image:: ../auto_examples/cluster/images/plot_lena_segmentation_1.png
    :target: ../auto_examples/cluster/plot_lena_segmentation.html
    :scale: 65

.. |lena_discretize| image:: ../auto_examples/cluster/images/plot_lena_segmentation_2.png
    :target: ../auto_examples/cluster/plot_lena_segmentation.html
    :scale: 65

Different label assignement strategies
---------------------------------------

Different label assignement strategies can be used, corresponding to the
`assign_labels` parameter of :class:`SpectralClustering`.
The `kmeans` strategie can match finer details of the data, but it can be
more unstable. In particular, unless you control the `random_state`, it
may not be reproducible from run-to-run, as it depends on a random
initialization. On the other hand, the `discretize` strategy is 100%
reproducible, but it tends to create parcels of fairly even and
geometrical shape.

=====================================  =====================================
 `assign_labels="kmeans"`               `assign_labels="discretize"`
=====================================  =====================================
|lena_kmeans|                          |lena_discretize|
=====================================  =====================================


.. topic:: References:

 * `"A Tutorial on Spectral Clustering"
   <http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.165.9323>`_
   Ulrike von Luxburg, 2007

 * `"Normalized cuts and image segmentation"
   <http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.160.2324>`_
   Jianbo Shi, Jitendra Malik, 2000

 * `"A Random Walks View of Spectral Segmentation"
   <http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.33.1501>`_
   Marina Meila, Jianbo Shi, 2001

 * `"On Spectral Clustering: Analysis and an algorithm"
   <http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.19.8100>`_
   Andrew Y. Ng, Michael I. Jordan, Yair Weiss, 2001


.. _hierarchical_clustering:

Hierarchical clustering
=======================

Hierarchical clustering is a general family of clustering algorithms that
build nested clusters by merging them successively. This hierarchy of
clusters represented as a tree (or dendrogram). The root of the tree is
the unique cluster that gathers all the samples, the leaves being the
clusters with only one sample. See the `Wikipedia page
<http://en.wikipedia.org/wiki/Hierarchical_clustering>`_ for more
details.

The :class:`Ward` object performs a hierarchical clustering based on
the Ward algorithm, that is a variance-minimizing approach. At each
step, it minimizes the sum of squared differences within all clusters
(inertia criterion).

This algorithm can scale to large number of samples when it is used jointly
with an connectivity matrix, but can be computationally expensive when no
connectivity constraints are added between samples: it considers at each step
all the possible merges.


Adding connectivity constraints
-------------------------------

An interesting aspect of the :class:`Ward` object is that connectivity
constraints can be added to this algorithm (only adjacent clusters can be
merged together), through an connectivity matrix that defines for each
sample the neighboring samples following a given structure of the data. For
instance, in the swiss-roll example below, the connectivity constraints
forbid the merging of points that are not adjacent on the swiss roll, and
thus avoid forming clusters that extend across overlapping folds of the
roll.

.. |unstructured| image:: ../auto_examples/cluster/images/plot_ward_structured_vs_unstructured_1.png
        :target: ../auto_examples/cluster/plot_ward_structured_vs_unstructured.html
        :scale: 49

.. |structured| image:: ../auto_examples/cluster/images/plot_ward_structured_vs_unstructured_2.png
        :target: ../auto_examples/cluster/plot_ward_structured_vs_unstructured.html
        :scale: 49

.. centered:: |unstructured| |structured|


The connectivity constraints are imposed via an connectivity matrix: a
scipy sparse matrix that has elements only at the intersection of a row
and a column with indices of the dataset that should be connected. This
matrix can be constructed from a-priori information, for instance if you
wish to cluster web pages, but only merging pages with a link pointing
from one to another. It can also be learned from the data, for instance
using :func:`sklearn.neighbors.kneighbors_graph` to restrict
merging to nearest neighbors as in the :ref:`swiss roll
<example_cluster_plot_ward_structured_vs_unstructured.py>` example, or
using :func:`sklearn.feature_extraction.image.grid_to_graph` to
enable only merging of neighboring pixels on an image, as in the
:ref:`Lena <example_cluster_plot_lena_ward_segmentation.py>` example.

.. topic:: Examples:

 * :ref:`example_cluster_plot_lena_ward_segmentation.py`: Ward clustering
   to split the image of lena in regions.

 * :ref:`example_cluster_plot_ward_structured_vs_unstructured.py`: Example of
   Ward algorithm on a swiss-roll, comparison of structured approaches
   versus unstructured approaches.

 * :ref:`example_cluster_plot_feature_agglomeration_vs_univariate_selection.py`:
   Example of dimensionality reduction with feature agglomeration based on
   Ward hierarchical clustering.

.. _dbscan:

DBSCAN
======

The :class:`DBSCAN` algorithm views clusters as areas of high density
separated by areas of low density. Due to this rather generic view, clusters
found by DBSCAN can be any shape, as opposed to k-means which assumes that
clusters are convex shaped. The central component to the DBSCAN is the concept
of *core samples*, which are samples that are in areas of high density. A
cluster is therefore a set of core samples, each highly similar to each other
and a set of non-core samples that are similar to a core sample (but are not
themselves core samples). There are two parameters to the algorithm,
`min_points` and `eps`, which define formally what we mean when we say *dense*.
A higher `min_points` or lower `eps` indicate higher density necessary to form
a cluster.

More formally, we define a core sample as being a sample in the dataset such
that there exists `min_samples` other samples with a similarity higher than
`eps` to it, which are defined as *neighbors* of the core sample. This tells
us that the core sample is in a dense area of the vector space. A cluster
is a set of core samples, that can be built by recursively by taking a core
sample, finding all of its neighbors that are core samples, finding all of
*their* neighbors that are core samples, and so on. A cluster also has a
set of non-core samples, which are samples that are neighbors of a core sample
in the cluster but are not themselves core samples. Intuitively, these samples
are on the fringes of a cluster.

Any core sample is part of a cluster, by definition. Further, any cluster has
at least `min_samples` points in it, following the definition of a core
sample. For any sample that is not a core sample, and does not have a
similarity higher than `eps` to a core sample, it is considered an outlier by
the algorithm.

The algorithm is non-deterministic, however the core samples themselves will
always belong to the same clusters (although the labels themselves may be
different). The non-determinism comes from deciding on which cluster a
non-core sample belongs to. A non-core sample can be have a similarity higher
than `eps` to two core samples in different classes. Following from the
triangular inequality, those two core samples would be less similar than
`eps` from each other -- else they would be in the same class. The non-core
sample is simply assigned to which ever cluster is generated first, where
the order is determined randomly within the code. Other than the ordering of,
the dataset, the algorithm is deterministic, making the results relatively
stable between iterations on the same data.

In the figure below, the color indicates cluster membership, with large circles
indicating core samples found by the algorithm. Smaller circles are non-core 
samples that are still part of a cluster. Moreover, the outliers are indicated
by black points below.

.. |dbscan_results| image:: ../auto_examples/cluster/images/plot_dbscan_1.png
        :target: ../auto_examples/cluster/plot_dbscan.html
        :scale: 50

.. centered:: |dbscan_results|

.. topic:: Examples:

 * :ref:`example_cluster_plot_dbscan.py`: Clustering synthetic data with DBSCAN

.. topic:: References:

 * "A Density-Based Algorithm for Discovering Clusters in Large Spatial Databases
   with Noise"
   Ester, M., H. P. Kriegel, J. Sander, and X. Xu,
   In Proceedings of the 2nd International Conference on Knowledge Discovery
   and Data Mining, Portland, OR, AAAI Press, pp. 226–231. 1996

.. _clustering_evaluation:

Clustering performance evaluation
=================================

Evaluating the performance of a clustering algorithm is not as trivial as
counting the number of errors or the precision and recall of a supervised
classification algorithm. In particular any evaluation metric should not
take the absolute values of the cluster labels into account but rather
if this clustering define separations of the data similar to some ground
truth set of classes or satisfying some assumption such that members
belong to the same class are more similar that members of different
classes according to some similarity metric.

.. currentmodule:: sklearn.metrics

Inertia
-------

Presentation and usage
~~~~~~~~~~~~~~~~~~~~~~

TODO: factorize inertia computation out of kmeans and then write me!


Advantages
~~~~~~~~~~

- No need for the ground truth knowledge of the "real" classes.

Drawbacks
~~~~~~~~~

- Inertia makes the assumption that clusters are convex and isotropic
  which is not always the case especially of the clusters are manifolds
  with weird shapes: for instance inertia is a useless metrics to evaluate
  clustering algorithm that tries to identify nested circles on a 2D plane.

- Inertia is not a normalized metrics: we just know that lower values are
  better and bounded by zero. One potential solution would be to adjust
  inertia for random clustering (assuming the number of ground truth classes
  is known).


Adjusted Rand index
-------------------

Presentation and usage
~~~~~~~~~~~~~~~~~~~~~~

Given the knowledge of the ground truth class assignments ``labels_true``
and our clustering algorithm assignments of the same samples
``labels_pred``, the **adjusted Rand index** is a function that measures
the **similarity** of the two assignments, ignoring permutations and **with
chance normalization**::

  >>> from sklearn import metrics
  >>> labels_true = [0, 0, 0, 1, 1, 1]
  >>> labels_pred = [0, 0, 1, 1, 2, 2]

  >>> metrics.adjusted_rand_score(labels_true, labels_pred)  # doctest: +ELLIPSIS
  0.24...

One can permute 0 and 1 in the predicted labels and rename `2` by `3` and get
the same score::

  >>> labels_pred = [1, 1, 0, 0, 3, 3]
  >>> metrics.adjusted_rand_score(labels_true, labels_pred)  # doctest: +ELLIPSIS
  0.24...

Furthermore, :func:`adjusted_rand_score` is **symmetric**: swapping the argument
does not change the score. It can thus be used as a **consensus
measure**::

  >>> metrics.adjusted_rand_score(labels_pred, labels_true)  # doctest: +ELLIPSIS
  0.24...

Perfect labeling is scored 1.0::

  >>> labels_pred = labels_true[:]
  >>> metrics.adjusted_rand_score(labels_true, labels_pred)
  1.0

Bad (e.g. independent labelings) have negative or close to 0.0 scores::

  >>> labels_true = [0, 1, 2, 0, 3, 4, 5, 1]
  >>> labels_pred = [1, 1, 0, 0, 2, 2, 2, 2]
  >>> metrics.adjusted_rand_score(labels_true, labels_pred)  # doctest: +ELLIPSIS
  -0.12...


Advantages
~~~~~~~~~~

- **Random (uniform) label assignments have a ARI score close to 0.0**
  for any value of ``n_clusters`` and ``n_samples`` (which is not the
  case for raw Rand index or the V-measure for instance).

- **Bounded range [-1, 1]**: negative values are bad (independent
  labelings), similar clusterings have a positive ARI, 1.0 is the perfect
  match score.

- **No assumption is made on the cluster structure**: can be used
  to compare clustering algorithms such as k-means which assumes isotropic
  blob shapes with results of spectral clustering algorithms which can
  find cluster with "folded" shapes.


Drawbacks
~~~~~~~~~

- Contrary to inertia, **ARI requires knowledge of the ground truth
  classes** while is almost never available in practice or requires manual
  assignment by human annotators (as in the supervised learning setting).

  However ARI can also be useful in a purely unsupervised setting as a
  building block for a Consensus Index that can be used for clustering
  model selection (TODO).


.. topic:: Examples:

 * :ref:`example_cluster_plot_adjusted_for_chance_measures.py`: Analysis of
   the impact of the dataset size on the value of clustering measures
   for random assignments.


Mathematical formulation
~~~~~~~~~~~~~~~~~~~~~~~~

If C is a ground truth class assignment and K the clustering, let us
define :math:`a` and :math:`b` as:

- :math:`a`, the number of pairs of elements that are in the same set
  in C and in the same set in K

- :math:`b`, the number of pairs of elements that are in different sets
  in C and in different sets in K

The raw (unadjusted) Rand index is then given by:

.. math:: \text{RI} = \frac{a + b}{C_2^{n_{samples}}}

Where :math:`C_2^{n_{samples}}` is the total number of possible pairs
in the dataset (without ordering).

However the RI score does not guarantee that random label assignments
will get a value close to zero (esp. if the number of clusters is in
the same order of magnitude as the number of samples).

To counter this effect we can discount the expected RI :math:`E[\text{RI}]` of
random labelings by defining the adjusted Rand index as follows:

.. math:: \text{ARI} = \frac{\text{RI} - E[\text{RI}]}{\max(\text{RI}) - E[\text{RI}]}

.. topic:: References

 * `Comparing Partitions
   <http://www.springerlink.com/content/x64124718341j1j0/>`_
   L. Hubert and P. Arabie, Journal of Classification 1985

 * `Wikipedia entry for the adjusted Rand index
   <http://en.wikipedia.org/wiki/Rand_index#Adjusted_Rand_index>`_


Mutual Information based scores
-------------------------------

Presentation and usage
~~~~~~~~~~~~~~~~~~~~~~

Given the knowledge of the ground truth class assignments ``labels_true`` and
our clustering algorithm assignments of the same samples ``labels_pred``, the
**Mutual Information** is a function that measures the **agreement** of the two
assignments, ignoring permutations.  Two different normalized versions of this
measure are available, **Normalized Mutual Information(NMI)** and **Adjusted
Mutual Information(AMI)**. NMI is often used in the literature while AMI was
proposed more recently and is **normalized against chance**::

  >>> from sklearn import metrics
  >>> labels_true = [0, 0, 0, 1, 1, 1]
  >>> labels_pred = [0, 0, 1, 1, 2, 2]

  >>> metrics.adjusted_mutual_info_score(labels_true, labels_pred)  # doctest: +ELLIPSIS
  0.22504...

One can permute 0 and 1 in the predicted labels and rename `2` by `3` and get
the same score::

  >>> labels_pred = [1, 1, 0, 0, 3, 3]
  >>> metrics.adjusted_mutual_info_score(labels_true, labels_pred)  # doctest: +ELLIPSIS
  0.22504...

All, :func:`mutual_info_score`, :func:`adjusted_mutual_info_score` and
:func:`normalized_mutual_info_score` are symmetric: swapping the argument does
not change the score. Thus they can be used as a **consensus measure**::

  >>> metrics.adjusted_mutual_info_score(labels_pred, labels_true)  # doctest: +ELLIPSIS
  0.22504...

Perfect labeling is scored 1.0::

  >>> labels_pred = labels_true[:]
  >>> metrics.adjusted_mutual_info_score(labels_true, labels_pred)
  1.0

  >>> metrics.normalized_mutual_info_score(labels_true, labels_pred)
  1.0

This is not true for ``mutual_info_score``, which is therefore harder to judge::

  >>> metrics.mutual_info_score(labels_true, labels_pred)  # doctest: +ELLIPSIS
  0.69...

Bad (e.g. independent labelings) have non-positive scores::

  >>> labels_true = [0, 1, 2, 0, 3, 4, 5, 1]
  >>> labels_pred = [1, 1, 0, 0, 2, 2, 2, 2]
  >>> metrics.adjusted_mutual_info_score(labels_true, labels_pred)  # doctest: +ELLIPSIS
  -0.10526...


Advantages
~~~~~~~~~~

- **Random (uniform) label assignments have a AMI score close to 0.0**
  for any value of ``n_clusters`` and ``n_samples`` (which is not the
  case for raw Mutual Information or the V-measure for instance).

- **Bounded range [0, 1]**:  Values close to zero indicate two label
  assignments that are largely independent, while values close to one
  indicate significant agreement. Further, values of exactly 0 indicate
  **purely** independent label assignments and a AMI of exactly 1 indicates
  that the two label assignments are equal (with or without permutation).

- **No assumption is made on the cluster structure**: can be used
  to compare clustering algorithms such as k-means which assumes isotropic
  blob shapes with results of spectral clustering algorithms which can
  find cluster with "folded" shapes.


Drawbacks
~~~~~~~~~

- Contrary to inertia, **MI-based measures require the knowledge
  of the ground truth classes** while almost never available in practice or
  requires manual assignment by human annotators (as in the supervised learning
  setting).

  However MI-based measures can also be useful in purely unsupervised setting as a
  building block for a Consensus Index that can be used for clustering
  model selection.

- NMI and MI are not adjusted against chance.


.. topic:: Examples:

 * :ref:`example_cluster_plot_adjusted_for_chance_measures.py`: Analysis of
   the impact of the dataset size on the value of clustering measures
   for random assignments. This example also includes the Adjusted Rand 
   Index.


Mathematical formulation
~~~~~~~~~~~~~~~~~~~~~~~~
Assume two label assignments (of the same data), :math:`U` with :math:`R`
classes and :math:`V` with :math:`C` classes. The entropy of either is the
amount of uncertaintly for an array, and can be calculated as:

.. math:: H(U) = \sum_{i=1}^{|R|}P(i)\log(P(i))

Where P(i) is the number of instances in U that are in class :math:`R_i`.
Likewise, for :math:`V`:

.. math:: H(V) = \sum_{j=1}^{|C|}P'(j)\log(P'(j))

Where P'(j) is the number of instances in V that are in class :math:`C_j`.

The mutual information between :math:`U` and :math:`V` is
calculated by:

.. math:: \text{MI}(U, V) = \sum_{i=1}^{|R|}\sum_{j=1}^{|C|}P(i, j)\log\left(\frac{P(i,j)}{P(i)P'(j)}\right)

Where P(i, j) is the number of instances with label :math:`R_i` 
and also with label :math:`C_j`.

The normalized mutual information is defined as

.. math:: \text{NMI}(U, V) = \frac{\text{MI}(U, V)}{\sqrt{H(U)H(V)}}

This value of the mutual information and also the normalized variant is not
adjusted for chance and will tend to increase as the number of different labels
(clusters) increases, regardless of the actual amount of "mutual information"
between the label assignments.

The expected value for the mutual information can be calculated using the
following equation, from Vinh, Epps, and Bailey, (2009). In this equation,
:math:`a_i` is the number of instances with label :math:`U_i` and
:math:`b_j` is the number of instances with label :math:`V_j`.


.. math:: E[\text{MI}(U,V)]=\sum_{i=1}^R \sum_{j=1}^C \sum_{n_{ij}=(a_i+b_j-N)^+
   }^{\min(a_i, b_j)} \frac{n_{ij}}{N}\log \left( \frac{ N.n_{ij}}{a_i b_j}\right)
   \frac{a_i!b_j!(N-a_i)!(N-b_j)!}{N!n_{ij}!(a_i-n_{ij})!(b_j-n_{ij})!
   (N-a_i-b_j+n_{ij})!}

Using the expected value, the adjusted mutual information can then be 
calculated using a similar form to that of the adjusted Rand index:

.. math:: \text{AMI} = \frac{\text{MI} - E[\text{MI}]}{\max(H(U), H(V)) - E[\text{MI}]}

.. topic:: References

 * Strehl, Alexander, and Joydeep Ghosh (2002). "Cluster ensembles – a
   knowledge reuse framework for combining multiple partitions". Journal of
   Machine Learning Research 3: 583–617. doi:10.1162/153244303321897735

 * Vinh, Epps, and Bailey, (2009). "Information theoretic measures 
   for clusterings comparison". Proceedings of the 26th Annual International
   Conference on Machine Learning - ICML '09.
   doi:10.1145/1553374.1553511. ISBN 9781605585161.

 * Vinh, Epps, and Bailey, (2010). Information Theoretic Measures for
   Clusterings Comparison: Variants, Properties, Normalization and
   Correction for Chance}, JMLR
   http://jmlr.csail.mit.edu/papers/volume11/vinh10a/vinh10a.pdf

 * `Wikipedia entry for the (normalized) Mutual Information
   <http://en.wikipedia.org/wiki/Mutual_Information>`_

 * `Wikipedia entry for the Adjusted Mutual Information
   <http://en.wikipedia.org/wiki/Adjusted_Mutual_Information>`_

Homogeneity, completeness and V-measure
---------------------------------------

Presentation and usage
~~~~~~~~~~~~~~~~~~~~~~

Given the knowledge of the ground truth class assignments of the samples,
it is possible to define some intuitive metric using conditional entropy
analysis.

In particular Rosenberg and Hirschberg (2007) define the following two
desirable objectives for any cluster assignment:

- **homogeneity**: each cluster contains only members of a single class.

- **completeness**: all members of a given class are assigned to the same
  cluster.

We can turn those concept as scores :func:`homogeneity_score` and
:func:`completeness_score`. Both are bounded below by 0.0 and above by
1.0 (higher is better)::

  >>> from sklearn import metrics
  >>> labels_true = [0, 0, 0, 1, 1, 1]
  >>> labels_pred = [0, 0, 1, 1, 2, 2]

  >>> metrics.homogeneity_score(labels_true, labels_pred)  # doctest: +ELLIPSIS
  0.66...

  >>> metrics.completeness_score(labels_true, labels_pred) # doctest: +ELLIPSIS
  0.42...

Their harmonic mean called **V-measure** is computed by
:func:`v_measure_score`::

  >>> metrics.v_measure_score(labels_true, labels_pred)    # doctest: +ELLIPSIS
  0.51...

The V-measure is actually equivalent to the mutual information (NMI)
discussed above normalized by the sum of the label entropies [B2011]_.

Homogeneity, completensess and V-measure can be computed at once using
:func:`homogeneity_completeness_v_measure` as follows::

  >>> metrics.homogeneity_completeness_v_measure(labels_true, labels_pred)
  ...                                                      # doctest: +ELLIPSIS
  (0.66..., 0.42..., 0.51...)

The following clustering assignment is slighlty better, since it is
homogeneous but not complete::

  >>> labels_pred = [0, 0, 0, 1, 2, 2]
  >>> metrics.homogeneity_completeness_v_measure(labels_true, labels_pred)
  ...                                                      # doctest: +ELLIPSIS
  (1.0, 0.68..., 0.81...)

.. note::

  :func:`v_measure_score` is **symmetric**: it can be used to evaluate
  the **agreement** of two independent assignments on the same dataset.

  This is not the case for :func:`completeness_score` and
  :func:`homogeneity_score`: both are bound by the relationship::

    homogeneity_score(a, b) == completeness_score(b, a)


Advantages
~~~~~~~~~~

- **Bounded scores**: 0.0 is as bad as it can be, 1.0 is a perfect score

- Intuitive interpretation: clustering with bad V-measure can be
  **qualitatively analyzed in terms of homogeneity and completeness**
  to better feel what 'kind' of mistakes is done by the assigmenent.

- **No assumption is made on the cluster structure**: can be used
  to compare clustering algorithms such as k-means which assumes isotropic
  blob shapes with results of spectral clustering algorithms which can
  find cluster with "folded" shapes.


Drawbacks
~~~~~~~~~

- The previously introduced metrics are **not normalized w.r.t. random
  labeling**: this means that depending on the number of samples,
  clusters and ground truth classes, a completely random labeling will
  not always yield the same values for homogeneity, completeness and
  hence v-measure. In particular **random labeling won't yield zero
  scores especially when the number of clusters is large**.

  This problem can safely be ignored when the number of samples is more
  than a thousand and the number of clusters is less than 10. **For
  smaller sample sizes or larger number of clusters it is safer to use
  an adjusted index such as the Adjusted Rand Index (ARI)**.

.. figure:: ../auto_examples/cluster/images/plot_adjusted_for_chance_measures_1.png
   :target: ../auto_examples/cluster/plot_adjusted_for_chance_measures.html
   :align: center
   :scale: 100

- These metrics **require the knowledge of the ground truth classes** while
  almost never available in practice or requires manual assignment by
  human annotators (as in the supervised learning setting).


.. topic:: Examples:

 * :ref:`example_cluster_plot_adjusted_for_chance_measures.py`: Analysis of
   the impact of the dataset size on the value of clustering measures
   for random assignments.


Mathematical formulation
~~~~~~~~~~~~~~~~~~~~~~~~

Homogeneity and completeness scores are formally given by:

.. math:: h = 1 - \frac{H(C|K)}{H(C)}

.. math:: c = 1 - \frac{H(K|C)}{H(K)}

where :math:`H(C|K)` is the **conditional entropy of the classes given
the cluster assignments** and is given by:

.. math:: H(C|K) = - \sum_{c=1}^{|C|} \sum_{k=1}^{|K|} \frac{n_{c,k}}{n}
          \cdot \log\left(\frac{n_{c,k}}{n_k}\right)

and :math:`H(C)` is the **entropy of the classes** and is given by:

.. math:: H(C) = - \sum_{c=1}^{|C|} \frac{n_c}{n} \cdot \log\left(\frac{n_c}{n}\right)

with :math:`n` the total number of samples, :math:`n_c` and :math:`n_k`
the number of samples respectively belonging to class :math:`c` and
cluster :math:`k`, and finally :math:`n_{c,k}` the number of samples
from class :math:`c` assigned to cluster :math:`k`.

The **conditional entropy of clusters given class** :math:`H(K|C)` and the
**entropy of clusters** :math:`H(K)` are defined in a symmetric manner.

Rosenberg and Hirschberg further define **V-measure** as the **harmonic
mean of homogeneity and completeness**:

.. math:: v = 2 \cdot \frac{h \cdot c}{h + c}

.. topic:: References

 .. [RH2007] `V-Measure: A conditional entropy-based external cluster evaluation
   measure <http://acl.ldc.upenn.edu/D/D07/D07-1043.pdf>`_
   Andrew Rosenberg and Julia Hirschberg, 2007

 .. [B2011] `Identication and Characterization of Events in Social Media
   <http://www.cs.columbia.edu/~hila/hila-thesis-distributed.pdf>`_, Hila
   Becker, PhD Thesis. 

.. _silhouette_coefficient:

Silhouette Coefficient
----------------------

Presentation and usage
~~~~~~~~~~~~~~~~~~~~~~

If the ground truth labels are not known, evaluation must be performed using
the model itself. The Silhouette Coefficient
(:func:`sklearn.metrics.silhouette_score`)
is an example of such an evaluation, where a
higher Silhouette Coefficient score relates to a model with better defined
clusters. The Silhouette Coefficient is defined for each sample and is composed
of two scores:

- **a**: The mean distance between a sample and all other points in the same
  class.

- **b**: The mean distance between a sample and all other points in the *next
  nearest cluster*.

The Silhoeutte Coefficient *s* for a single sample is then given as:

.. math:: s = \frac{b - a}{max(a, b)}

The Silhouette Coefficient for a set of samples is given as the mean of the 
Silhouette Coefficient for each sample.


  >>> from sklearn import metrics
  >>> from sklearn.metrics import pairwise_distances
  >>> from sklearn import datasets
  >>> dataset = datasets.load_iris()
  >>> X = dataset.data
  >>> y = dataset.target

In normal usage, the Silhouette Coefficient is applied to the results of a
cluster analysis.

  >>> import numpy as np
  >>> from sklearn.cluster import KMeans
  >>> kmeans_model = KMeans(n_clusters=3, random_state=1).fit(X)
  >>> labels = kmeans_model.labels_
  >>> metrics.silhouette_score(X, labels, metric='euclidean')  
  ...                                                      # doctest: +ELLIPSIS
  0.55...

.. topic:: References

 * Peter J. Rousseeuw (1987). "Silhouettes: a Graphical Aid to the
   Interpretation and Validation of Cluster Analysis". Computational
   and Applied Mathematics 20: 53–65. doi:10.1016/0377-0427(87)90125-7.


Advantages
~~~~~~~~~~

- The score is bounded between -1 for incorrect clustering and +1 for highly
  dense clustering. Scores around zero indicate overlapping clusters.

- The score is higher when clusters are dense and well separated, which relates
  to a standard concept of a cluster.


Drawbacks
~~~~~~~~~

- The Silhouette Coefficient is generally higher for convex clusters than other
  concepts of clusters, such as density based clusters like those obtained
  through DBSCAN.


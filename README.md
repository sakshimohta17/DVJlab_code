# DVJlab_code
This file demonstrates the methodology behind a computational cardiac pipeline applied to 20 GMSH tetrahedral surface meshes spanning 9 subjects (identifiers: W282, W283, Y283, Y322, Y325, Y329, Y339, Y340, and Z210) across four time points: Weeks 0, 4, 8, and 12. Each mesh encodes the rat biventricular geometry with four labeled physical surfaces: left ventricular (LV) endocardium, right ventricular (RV) endocardium, epicardium, and basal plane.
The primary outputs are (1) LV and RV endocardial cavity volumes in mm³, (2) LV and RV free wall thicknesses in mm, and (3) interventricular septal thickness (mm)

1) Mesh Format and Parsing:
The  mesh file format encodes: Physical Names block: named surface/volume groups (base, epi, lv, rv, Group_1); Entities block: maps entity tags to physical group tags for both 2D surfaces and 3D volumes. Nodes block: 3D Cartesian coordinates (x, y, z) in millimetres for all mesh nodes); Elements block: connectivity for surface triangles (element type 2, 3-node) and volume tetrahedra (element type 4, 4-node). The parser resolves the two-level indirection (entity tag → physical tag → name) to label each element with its anatomical surface. The result is four named triangle lists (lv, rv, epi, base) and one tetrahedral list (Group_1) used downstream.

2) Quantification:
a) Endocardial Volume Computation:
Cavity volumes are computed from the triangulated endocardial surfaces using the divergence theorem (also known as the signed tetrahedral decomposition from the origin):
V = (1/6) | Σ v₁ · (v₂ × v₃) | where v₁, v₂, v₃ are the three vertices of each surface triangle, and the summation runs over all triangles in the closed endocardial surface. The absolute value ensures positive volume regardless of triangle orientation. This method is exact for polyhedral surfaces and equivalent to integrating the signed volume of tetrahedra formed between each triangle and the coordinate origin.
For the total myocardial volume, the sum of all tetrahedral element volumes is computed using the scalar triple product:
V_tet = |det([b−a, c−a, d−a])| / 6 where a, b, c, and d are the four vertices of the tetrahedron.
References: Lorensen & Cline (1987), Marching Cubes; Zhang & Chen (2001), Efficient Feature Extraction; Legrice et al., rat heart geometry studies.

b) Wall Thickness (Closest-Point Projection) :
Wall thickness is measured using the closest-point (nearest-neighbour) projection method: for each node on the endocardial surface, the minimum Euclidean distance to any node on the epicardial surface is computed:
d(p) = minᵢ ‖p − eᵢ‖₂,   thickness = mean{d(p)}
where p iterates over all endocardial surface nodes, and eᵢ iterates over all epicardial surface nodes. Outliers are excluded by rejecting distances outside the range [0.05, 20] mm.

Three thickness values are reported per mesh:
•	LV free wall: LV endocardial nodes (non-septal) → epicardial surface

•	RV free wall: RV endocardial nodes → epicardial surface

•	Interventricular septum: LV endocardial nodes in the septal region → RV endocardial surface
References: Bai et al. (2015)—JCMR—closest-point wall thickness in cardiac MRI; Haddad et al. (2008)—RV thickness methodology.

c) Interventricular Septum Identification:
The interventricular septum is identified geometrically as the spatial overlap region between the LV and RV endocardial bounding boxes, expanded by a 1 mm margin to capture the full junctional zone:
•	Compute axis-aligned bounding boxes for all LV and RV endocardial nodes

•	The septal region is defined as the intersection of these boxes (± 1 mm margin)

•	LV endocardial nodes falling within this region are classified as septal; all others as LV free wall

•	Septal thickness = closest-point distance from septal LV nodes to the RV endocardial surface

This spatial overlap approach is robust to inter-subject anatomical variation and avoids the need for explicit septal landmark annotation. It has been validated in biventricular rat models by Healy et al. (2011) and Young & Cowan (2012).

The pipeline is implemented in pure Python 3 with NumPy as the sole runtime dependency (matplotlib is optional for visualization). 
Key implementation properties:
•	Zero external mesh libraries: all parsing and geometry is native Python/NumPy

•	Spatial hash grid for O(n) nearest-neighbour queries (cell size = 1.5 mm, 3-cell search radius)

•	Outlier rejection: distances < 0.05 mm or > 20 mm excluded from thickness statistics

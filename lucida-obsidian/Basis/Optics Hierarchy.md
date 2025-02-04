Here's the hierarchy structure which is identical to the [[Arrow Library]] hierarchy and [[Monocle Library]] too
![[Pic. Optics Hierarchy]]

- [x] #TODO Is `Optional` the same as `AffineTraversal`? or not? ✅ 2025-02-04
	> Yes, an `Optional` is the same as `AffineTraversal`
- [x] #TODO where does such weird beast as `Grate` located? ✅ 2025-02-04
	> See [[Grates]]

The Importance of hierarchy is to specify what type does composition of optics should be. 
- Like if you compose Lenses you still get Lenses 
- or how you compose Lens and Traversal and get Traversal
- Some things are not composable
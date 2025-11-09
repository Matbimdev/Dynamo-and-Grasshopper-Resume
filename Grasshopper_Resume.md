# Revit Structures Grasshopper Definitions

These `.ghx` definitions are XML-based Grasshopper archives exported from Revit 2023 workflows, so they remain diffable and fully readable in Git rather than opaque binaries.【F:grasshopper/revit_structures/R23_Formwork.ghx†L1-L19】

## Contents

### `R23-Structure.ghx`
- Uses **Query Categories** to pull structural disciplines and category types from the active Revit model before downstream filtering.【F:grasshopper/revit_structures/R23-Structure.ghx†L140-L220】
- Relies on a persistent **Value Picker** to choose the specific categories that should pass through, followed by **Stream Filter** components to route each structural group (columns, beams, walls, slabs, foundations).【F:grasshopper/revit_structures/R23-Structure.ghx†L395-L520】

**Opportunities for improvement**
- Replace the Value Picker with a dynamic Category picker list (e.g., pulling names from the `Query Categories` output) so the routine responds automatically to model changes.【F:grasshopper/revit_structures/R23-Structure.ghx†L140-L520】
- Promote key filters to named inputs or clusters with visible notes, giving downstream users a quick understanding of which disciplines and types are expected without inspecting hidden wires.【F:grasshopper/revit_structures/R23-Structure.ghx†L140-L220】

### `R23_EST_Takeoff.ghx`
- Mirrors the category query + Value Picker approach to organize structural take-off packages before calculating parameters for schedules.【F:grasshopper/revit_structures/R23_EST_Takeoff.ghx†L140-L470】
- Ends with a `Revit Create Schedule` GhPython component that opens a transaction, builds schedule views, and injects fields according to the selected parameters for each structural item.【F:grasshopper/revit_structures/R23_EST_Takeoff.ghx†L6005-L6234】

**Opportunities for improvement**
- Drive the schedule category and field selection from data trees instead of a static Value Picker so that new categories do not require editing persistent values.【F:grasshopper/revit_structures/R23_EST_Takeoff.ghx†L140-L470】
- Surface the GhPython inputs (`Create`, `Title`, `Category`, `Fields`) as documented panels or clusters so collaborators understand the required data structures before triggering schedule creation.【F:grasshopper/revit_structures/R23_EST_Takeoff.ghx†L6187-L6234】

### `R23_Formwork.ghx`
- Starts by **Deconstructing Breps** from the structural solids and **Evaluating Surfaces** at UV coordinates to gather normals and orient area calculations for each face.【F:grasshopper/revit_structures/R23_Formwork.ghx†L124-L320】
- Subsequent clusters propagate those surface evaluations into grouped outputs that can be exported back to Revit for reporting (faces, normals, plane data).【F:grasshopper/revit_structures/R23_Formwork.ghx†L124-L320】

**Opportunities for improvement**
- Unhide or wrap the core `Deconstruct Brep`/`Evaluate Surface` steps in annotated clusters so contributors do not accidentally delete them while editing the definition.【F:grasshopper/revit_structures/R23_Formwork.ghx†L124-L320】
- Factor the repeated area calculations into reusable clusters to reuse the workflow across projects without copying large node groups.【F:grasshopper/revit_structures/R23_Formwork.ghx†L124-L320】

### `R23_Formwork_Area.ghx`
- References Rhino geometry layer-by-layer to pull elements that represent the formwork surfaces to be measured.【F:grasshopper/revit_structures/R23_Formwork_Area.ghx†L130-L179】
- Computes area totals per element before running a **Collision One|Many** check that flags interfering formwork pieces for coordination.【F:grasshopper/revit_structures/R23_Formwork_Area.ghx†L234-L441】

**Opportunities for improvement**
- Replace the hard-coded `Reference by Layer` inputs with parameterized lists or Data Inputs so renaming Rhino layers does not break the workflow.【F:grasshopper/revit_structures/R23_Formwork_Area.ghx†L130-L300】
- Expose tolerance controls for the collision component to tune clash sensitivity depending on model LOD and avoid false positives.【F:grasshopper/revit_structures/R23_Formwork_Area.ghx†L357-L441】

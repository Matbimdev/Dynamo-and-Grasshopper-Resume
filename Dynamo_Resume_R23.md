# Revit 2023 branch

This README centralizes documentation for Dynamo graphs that target Revit 2023 (Dynamo 2.17.1). Each section records the discipline, intent, dependencies, implementation details, and follow-up items for the corresponding `.dyn` workspace so future updates can be scoped quickly.

## R23_ARC_CE_CeilingByWalls.dyn
- **Discipline**: Architecture – ceilings and interior coordination.
- **Purpose**: Builds ceiling elements that trace the interior faces of selected walls, offsets them vertically, and locks the generated references for coordination.

### Inputs
- **Wall selection**: `Select Model Elements` configured for the `OST_Walls` category, storing the 16 wall instances referenced in the workspace.
- **Ceiling type**: `Ceiling Types` drop-down returning a single Revit ceiling type element.
- **Level**: `Levels` drop-down that provides the hosting level for the new ceilings.
- **Height offset**: Numeric input (`Double`) with a default value of 2.7 project units.
- **Lock references toggle**: Boolean input that defaults to `True` and drives the downstream locking helpers.

### Outputs
- **Ceiling elements** created through `Rhythm.Revit.Elements.Ceiling.ByCurveLoops`.
- **Height offset parameter** written with `Element.SetParameterByName` (language-aware string derived from the document language Python node).
- **Reference locks** applied through Genius Loci custom nodes to maintain relationships between source walls and generated geometry.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- Genius Loci — version 2023.2.21
- archi-lab.net — version 2023.213.1223
- Rhythm — version 2023.1.1

### Python nodes
1. **Dependent curve collector** – unwraps the selected wall elements, gathers `CurveElement` dependents, and returns nested lists for downstream curve grouping.
2. **Document language probe** – retrieves `doc.Application.Language` to toggle between English and Spanish parameter names before writing offsets.

### Graph metrics
- 129 nodes
- 151 connectors
- 0 code block nodes
- 7 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Upgrade Python engine** – Port the IronPython scripts to CPython 3 to stay compatible with Revit 2025+ and Dynamo 3.x.
2. **Expose units metadata** – Document the expected unit for the 2.7 offset (e.g., metres vs. feet) so users do not apply unintended elevations.
3. **Parameter name map** – Replace the two-option language check with a dictionary-based lookup that supports additional locales without editing code.
4. **Reduce manual selection** – Replace the baked-in wall instance IDs with a category or view filter selection to avoid stale references when the Revit model changes.
5. **Watch node cleanup** – Swap production watch nodes for custom logging or notes to keep performance predictable in large projects.

### Next steps
- Validate the graph in Revit 2024 to confirm the Rhythm and Clockwork nodes remain compatible or document required updates.
- Publish sample datasets demonstrating expected wall configurations and ceiling results for QA.
- Mirror this documentation style across additional Dynamo graphs as they are added to the branch.

## R23-25_GEN_SEC_Move_and_Zoom.dyn
- **Discipline**: Coordination / General Modeling Support.
- **Purpose**: Recenters paired section or elevation views around selected elements, applies user-defined offsets, and zooms the active Revit view for both Revit 2023 and 2025 projects.

### Inputs
- **View selector**: Drop-down listing section and elevation views, routed through `Clockwork.View.GetByName`.
- **Target elements**: `Select Model Elements` node that feeds bounding boxes to the move routine.
- **Offset vector**: `Vector.ByCoordinates` controls X/Y/Z displacements before the zoom step.
- **Zoom factor**: Number slider (0.5 – 2.0) applied to the `UIView.ZoomAndCenterRectangle` call in Python.
- **Reorient toggle**: Boolean switch that optionally aligns the section box perpendicular to the average normal of the selected elements.

### Outputs
- **Relocated view** returned as a single item so downstream scripts can chain additional overrides.
- **Zoom confirmation text** displayed through a watch node for user feedback.
- **Error report list** capturing elements skipped because they were locked or on sheets.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- Genius Loci — version 2023.2.21
- BimorphNodes — version 5.2.2

### Python nodes
1. **UIView resolver** – Grabs the active `UIView` and wraps `ZoomAndCenterRectangle` so Dynamo can set the camera programmatically.
2. **Section box aligner** – Calculates a best-fit plane for the selected elements and rotates the section box when the toggle is enabled.

### Graph metrics
- 64 nodes
- 78 connectors
- 1 code block node
- 4 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Sheet-awareness** – Detect whether the chosen view is placed on a sheet and prompt the user before moving it.
2. **Saved configurations** – Persist common offset/zoom presets in a JSON file so coordinators can reuse them.
3. **3D support** – Extend the Python helpers to work with 3D views by exposing the `SectionBox` object when applicable.

### Next steps
- Validate behaviour in Revit 2025 to confirm UI automation remains stable under the new API.
- Wrap the zoom routine into a custom node for reuse in other navigation scripts.

## R23_ARC_CE_CeilingInside.dyn
- **Discipline**: Architecture – interior finishes.
- **Purpose**: Generates ceilings that follow the room finish boundary for selected rooms, offsetting the sketch to stay within the interior envelope.

### Inputs
- **Room selection**: `Select Model Elements` filtered to `OST_Rooms`.
- **Ceiling type**: Drop-down retrieving the desired type via `Clockwork.CeilingType.ByName`.
- **Host level**: `Levels` selector that establishes the creation plane.
- **Perimeter offset**: Number input that shrinks the boundary relative to the room finish lines.
- **Sketch cleanup toggle**: Boolean input that decides whether to dissolve redundant line segments using a Python routine.

### Outputs
- **Room-based ceilings** produced with `Rhythm.Ceilings.ByRoom`.
- **Offset parameter updates** for `Height Offset From Level`.
- **QA summary** list showing rooms that failed due to non-closed boundaries.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- Rhythm — version 2023.1.1
- Genius Loci — version 2023.2.21

### Python nodes
1. **Boundary extractor** – Pulls the finish boundary from each room and converts it to planar curves.
2. **Segment merger** – Simplifies polylines by removing colinear segments prior to ceiling creation.
3. **Failure logger** – Captures `Ceiling.Create` exceptions and formats messages for the QA summary.

### Graph metrics
- 141 nodes
- 168 connectors
- 2 code block nodes
- 6 watch nodes
- 3 IronPython nodes

### Opportunities for improvement
1. **Room filters** – Allow users to filter by department or level rather than manual selection.
2. **Composite offsets** – Support different offsets for perimeter and inner islands to handle columns.
3. **Analytics** – Record processed room counts and duration to a shared log for production monitoring.

### Next steps
- Test with spaces as an alternative input for MEP coordination models.
- Package the segment merger into a reusable custom node shared across floor and ceiling scripts.

## R23_ARC_FL_FloorByWalls.dyn
- **Discipline**: Architecture – floor finishing.
- **Purpose**: Creates floor elements by tracing the core faces of selected walls and applying optional structural parameters.

### Inputs
- **Wall set**: `Select Model Elements` restricted to wall instances.
- **Floor type**: Drop-down retrieving a system floor type via `Clockwork.FloorType.ByName`.
- **Creation level**: `Levels` input establishing the base plane.
- **Height offset**: Numeric value controlling the z-offset from the level.
- **Structural toggle**: Boolean to set the `Structural` parameter on the new floors.

### Outputs
- **Floor elements** generated with `Floor.ByOutlineTypeAndLevel` (Rhythm wrapper).
- **Parameter sync** for `Structural` and `Comments` fields based on user input.
- **Warning list** detailing walls that produced open loops or overlapping sketches.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- Rhythm — version 2023.1.1
- Genius Loci — version 2023.2.21
- SpringNodes — version 210.0.1

### Python nodes
1. **Profile builder** – Collects wall location curves, applies offsets, and stitches them into closed loops.
2. **Parameter writer** – Uses the Revit API to set boolean and text parameters not exposed by out-of-the-box nodes.
3. **Loop validator** – Flags self-intersecting profiles before sending them to the floor creation node.

### Graph metrics
- 136 nodes
- 159 connectors
- 3 code block nodes
- 5 watch nodes
- 3 IronPython nodes

### Opportunities for improvement
1. **Curve caching** – Cache wall curves per level to accelerate reruns on large models.
2. **Design options** – Detect elements hosted on secondary design options and prompt the user to switch before creation.
3. **Profile visualization** – Add a `DirectShape` preview to review sketches prior to committing floors.

### Next steps
- Compare results with the Revit 2025 API to confirm no geometry kernel regressions.
- Consolidate shared curve logic with the ceiling scripts to avoid duplication.

## R23_ARC_WA_GetFinishFace.dyn
- **Discipline**: Architecture – wall finishing analysis.
- **Purpose**: Identifies the finish side surface of selected walls and returns face geometry and orientation metadata for downstream automation.

### Inputs
- **Wall selection**: `Select Model Elements` targeting wall instances.
- **Finish side toggle**: Boolean to switch between interior/exterior faces.
- **Offset distance**: Number slider that inflates the face by a small tolerance for clash detection.
- **Return normal toggle**: Boolean controlling whether to output surface normals for each face.

### Outputs
- **Face surfaces** represented as Dynamo surfaces for preview and documentation.
- **Face normals** as vectors when requested.
- **Wall identifiers** (element ID and mark) paired with the surfaces.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- Genius Loci — version 2023.2.21

### Python nodes
1. **Face resolver** – Uses `HostObjectUtils.GetSideFaces` to pull the correct face references.
2. **Normal calculator** – Samples UV coordinates on the surface and returns orientation vectors.

### Graph metrics
- 74 nodes
- 88 connectors
- 1 code block node
- 3 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Support curved walls** – Add logic for segmented curtain walls which currently return partial faces.
2. **Batch export** – Stream the resulting surfaces to a DWG for external coordination tools.
3. **Unit handling** – Normalize offset distances to project units selected at runtime.

### Next steps
- Convert the output package into a shared custom node used by wall finish placement workflows.
- Introduce automated QA checks that compare finish side orientation against room-facing directions.

## R23_ARC_WA_WallFinish.dyn
- **Discipline**: Architecture – finishing automation.
- **Purpose**: Places finish walls or parts aligned with host walls, offsetting by finish thickness and tagging results.

### Inputs
- **Base wall set**: `Select Model Elements` for walls requiring finish treatments.
- **Finish type**: Drop-down of wall types flagged as finishes.
- **Offset thickness**: Numeric input defining gap between structural and finish walls.
- **Height override**: Optional number to override the host wall top constraint.
- **Tag on placement toggle**: Boolean that triggers automatic tagging of the generated walls.

### Outputs
- **Finish wall elements** created via a Python helper using `Wall.Create`.
- **Assigned parameters** for `Comments`, `Type Mark`, and `Finish Code`.
- **Tag elements** placed in the active view when the toggle is true.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- archi-lab.net — version 2023.213.1223
- Genius Loci — version 2023.2.21

### Python nodes
1. **Finish creator** – Copies host wall location curves, applies offsets, and creates the finish wall instances.
2. **Parameter mapper** – Synchronises mark and comments between host and finish walls.
3. **Tagger** – Places independent tags on the newly created walls using the active view context.

### Graph metrics
- 162 nodes
- 194 connectors
- 2 code block nodes
- 6 watch nodes
- 3 IronPython nodes

### Opportunities for improvement
1. **Automatic joins** – Integrate join/ungroup rules to avoid manual cleanup at corners.
2. **Type validation** – Check that the selected finish type has matching height and structure before creation.
3. **Batch tagging** – Allow users to tag existing finish walls without recreating them.

### Next steps
- Provide sample wall assemblies to document expected finish offsets.
- Explore packaging the creator routine as a custom node for use across branches.

## R23_COO_CLASH_Import_and_ViewList.dyn
- **Discipline**: Coordination – clash detection.
- **Purpose**: Imports Navisworks clash test results, writes metadata to Revit elements, and compiles a view list for follow-up review.

### Inputs
- **CSV path**: `File Path` input referencing the exported Navisworks clash report.
- **View template**: `View Templates` drop-down for the 3D views generated per clash.
- **Discipline filter**: String input used to filter clash groups by discipline code.
- **Group-by parameter**: Text input referencing the shared parameter used to cluster clashes.
- **Create sheets toggle**: Boolean that decides whether to create placeholder sheets for high-priority clashes.

### Outputs
- **Parameter updates** applied to involved elements (`Clash_ID`, `Clash_Status`).
- **3D views** generated and collected into a list for documentation.
- **View schedule** that lists the created views when the toggle is enabled.

### Packages
- BimorphNodes — version 5.2.2
- Clockwork for Dynamo 2.x — version 2.12.3
- Data-Shapes — version 2023.2.1

### Python nodes
1. **Clash parser** – Reads the CSV, groups clashes, and outputs dictionaries keyed by test name.
2. **View builder** – Creates 3D views with applied section boxes and templates, returning view IDs for scheduling.

### Graph metrics
- 119 nodes
- 142 connectors
- 2 code block nodes
- 5 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Priority weighting** – Interpret Navisworks `Status`/`Approval` fields to auto-rank clashes.
2. **Bi-directional sync** – Push resolution status back to Navisworks via BCF export.
3. **Error resilience** – Add guardrails for missing parameters or renamed CSV columns.

### Next steps
- Pilot the workflow with mechanical and electrical teams to validate discipline filtering.
- Automate email summaries after view generation for nightly coordination reports.

## R23_ELE_CON_Conduits_ByCAD.dyn
- **Discipline**: Electrical – conduit routing.
- **Purpose**: Converts CAD polylines into Revit conduits, applying type, size, and offset rules per layer.

### Inputs
- **CAD link selection**: `Select Model Element` targeting the DWG link.
- **Layer-to-type map**: `Data-Shapes.UI.MultipleInputForm++` returning dictionaries of DWG layers and conduit types.
- **Conduit size override**: Number input specifying diameter when the map leaves it blank.
- **Elevation offset**: Numeric input applied to the Z coordinate of created conduits.
- **Slope toggle**: Boolean indicating whether to calculate slopes from polyline elevations.

### Outputs
- **Conduit elements** placed via `MEPover.Conduit.ByCurve`.
- **Size parameter updates** using `Element.SetParameterByName`.
- **Import log** summarizing created lengths per layer.

### Packages
- MEPover — version 1.6.0
- Clockwork for Dynamo 2.x — version 2.12.3
- BimorphNodes — version 5.2.2
- Data-Shapes — version 2023.2.1

### Python nodes
1. **Polyline extractor** – Pulls 3D polylines from the DWG, repairing short segments.
2. **Layer dispatcher** – Routes polylines to the correct conduit type based on the user map.
3. **Slope calculator** – Evaluates start/end Z to set slope parameters when enabled.

### Graph metrics
- 187 nodes
- 223 connectors
- 4 code block nodes
- 8 watch nodes
- 3 IronPython nodes

### Opportunities for improvement
1. **Unit normalization** – Detect DWG units automatically to avoid manual overrides.
2. **Batch validation** – Highlight polylines that self-intersect before attempting conduit creation.
3. **Performance tuning** – Replace chained `List.Map` nodes with custom nodes for large DWGs.

### Next steps
- Document standard layer naming conventions alongside the map input.
- Share sample CAD files for onboarding new electrical designers.

## R23_ELE_CON_Conduits_Fitting_ByCAD.dyn
- **Discipline**: Electrical – conduit detailing.
- **Purpose**: Places elbows and tees along imported conduit paths based on CAD geometry and bend thresholds.

### Inputs
- **Source conduits**: List of conduit elements created by the previous workflow.
- **Fitting family type**: Drop-down referencing electrical conduit fittings.
- **Bend threshold**: Angle value (degrees) that determines when to insert an elbow vs. continue straight.
- **Spacing override**: Optional number for fixed fitting spacing along straight runs.
- **Orient to slope toggle**: Boolean controlling whether fittings inherit conduit slopes.

### Outputs
- **Fitting elements** created via `MEPover.ConduitFitting.ByElements` Python helper.
- **Exception list** describing joints skipped due to insufficient length.
- **Summary table** counting fittings by type.

### Packages
- MEPover — version 1.6.0
- Genius Loci — version 2023.2.21
- Clockwork for Dynamo 2.x — version 2.12.3

### Python nodes
1. **Bend analyzer** – Computes angles between conduit segments and decides which fitting to place.
2. **Placement engine** – Calls the Revit API to insert fittings and orient connectors to match slopes.

### Graph metrics
- 131 nodes
- 154 connectors
- 2 code block nodes
- 5 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Dynamic fitting selection** – Allow per-layer fitting overrides similar to conduit types.
2. **Collision preview** – Generate temporary solids to highlight potential clashes before committing fittings.
3. **Reporting** – Export the summary table to Excel for prefabrication teams.

### Next steps
- Link the script to fabrication spools once QA is complete.
- Add a recovery routine to delete fittings if the CAD alignment changes.

## R23_ELE_DEV_CADText_to_Parameter.dyn
- **Discipline**: Electrical – device data management.
- **Purpose**: Reads CAD text annotations and writes the values into matching Revit device parameters.

### Inputs
- **CAD text selection**: `Select Model Elements` limited to text entities in the DWG link.
- **Target family category**: Drop-down to filter device instances to update.
- **Parameter name**: Text input specifying which instance parameter receives the value.
- **Layer filter**: String list to restrict text entities processed.
- **Offset distance**: Number controlling search radius between CAD text and Revit elements.

### Outputs
- **Parameter updates** applied to the matching devices.
- **Unmatched report** listing CAD texts that failed to find a host element.
- **Audit list** of elements whose values changed during the run.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- BimorphNodes — version 5.2.2
- Data-Shapes — version 2023.2.1

### Python nodes
1. **Spatial matcher** – Searches for Revit elements within the offset radius of each CAD text location.
2. **Parameter writer** – Sets the specified parameter and handles value conversions (numbers vs. strings).

### Graph metrics
- 92 nodes
- 111 connectors
- 1 code block node
- 4 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Bi-directional sync** – Optionally push existing Revit values back to CAD for verification.
2. **Parameter mapping table** – Support multiple parameters at once using a dictionary input.
3. **Tolerance preview** – Visualize the search radius in the Revit view before executing.

### Next steps
- Validate with bilingual parameter names to ensure the writer handles localization.
- Package the matcher routine for reuse in plumbing and fire protection annotation imports.

## R23_FP_SPR_Connect_TeeOrTakeoff.dyn
- **Discipline**: Fire Protection – sprinkler distribution.
- **Purpose**: Connects sprinkler heads to mains using either tee fittings or mechanical takeoffs depending on spacing and slope.

### Inputs
- **Main pipe selection**: `Select Model Elements` limited to pipe mains.
- **Branch family type**: Drop-down referencing tee or takeoff families.
- **Connector size**: Number input that overrides branch size when required.
- **Use takeoffs toggle**: Boolean deciding between tee and takeoff placement.
- **Slope parameter**: Number controlling vertical offsets for heads relative to mains.

### Outputs
- **Branch pipes** created via `MEPover.Pipe.ByPoints`.
- **Fittings** inserted using `MEPover.PipeFitting.ByElements`.
- **QA list** highlighting heads that could not connect due to distance or orientation.

### Packages
- MEPover — version 1.6.0
- Rhythm — version 2023.1.1
- Clockwork for Dynamo 2.x — version 2.12.3

### Python nodes
1. **Connector resolver** – Finds the closest connector on the main pipe for each sprinkler head.
2. **Branch builder** – Generates points for branch pipes and chooses between tee/takeoff families.
3. **Failure handler** – Logs elements that fail connection attempts for manual review.

### Graph metrics
- 168 nodes
- 201 connectors
- 3 code block nodes
- 7 watch nodes
- 3 IronPython nodes

### Opportunities for improvement
1. **Hydraulic sizing** – Integrate hydraulic calculations to suggest connector sizes automatically.
2. **Batch slopes** – Allow different slope values per branch zone.
3. **Prefabrication export** – Generate fabrication parts or spool drawings after connections succeed.

### Next steps
- Test with dry and wet system templates to confirm compatibility.
- Add warning notifications when mains already host a tee to avoid duplicates.

## R23_GEN_FAM_FlipWorkPlane.dyn
- **Discipline**: General – family management.
- **Purpose**: Flips the work plane orientation of selected hosted families (e.g., lighting fixtures) and optionally mirrors them.

### Inputs
- **Family instances**: `Select Model Elements` filtered by category.
- **Flip axis selector**: Drop-down listing available orientation axes.
- **Mirror toggle**: Boolean controlling whether to mirror the instance after flipping.
- **Keep constraints toggle**: Boolean to preserve host constraints when available.

### Outputs
- **Modified family instances** with updated `HandFlipped`/`FaceFlipped` status.
- **Result summary** reporting which instances could not flip due to host restrictions.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- Genius Loci — version 2023.2.21

### Python nodes
1. **Flip executor** – Calls `FamilyInstance.flipHand/flipFacing` methods safely and reports results.

### Graph metrics
- 56 nodes
- 63 connectors
- 1 code block node
- 2 watch nodes
- 1 IronPython node

### Opportunities for improvement
1. **Category presets** – Provide predefined sets for lighting, devices, and mechanical equipment.
2. **Transaction grouping** – Combine flips into fewer transactions to speed up large selections.
3. **Undo script** – Generate a list of instance IDs that were flipped to ease manual rollback if needed.

### Next steps
- Validate across hosted and face-based families to ensure host constraints persist.
- Explore packaging as a custom node shared between Revit 2022–2025 branches.

## R23_GEN_PRM_CADText_to_Parameter.dyn
- **Discipline**: General – parameter management.
- **Purpose**: Generalized version of the CAD text import script that supports any category/parameter mapping defined in an Excel table.

### Inputs
- **Excel mapping path**: `File Path` input referencing the mapping spreadsheet.
- **CAD link**: `Select Model Element` pointing to the DWG import.
- **Category filter**: List input containing Revit categories to update.
- **Search radius**: Number controlling how far from CAD text to search for elements.
- **Fallback value**: Text input used when no matching CAD text is found.

### Outputs
- **Parameter updates** applied per the Excel mapping.
- **Audit worksheet** optionally written back to Excel summarizing changes.
- **Exception list** capturing rows that failed due to missing parameters.

### Packages
- Data-Shapes — version 2023.2.1
- Clockwork for Dynamo 2.x — version 2.12.3
- BimorphNodes — version 5.2.2

### Python nodes
1. **Excel reader** – Loads the mapping table and converts it to Dynamo dictionaries.
2. **Updater** – Iterates through the mapped categories, finds nearby elements, and writes parameter values.

### Graph metrics
- 101 nodes
- 126 connectors
- 2 code block nodes
- 4 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Conflict resolution** – When multiple CAD texts map to one element, prompt the user to choose the preferred value.
2. **Unit conversions** – Detect numeric parameters and convert units automatically.
3. **Logging** – Append change logs to a shared CSV for auditing purposes.

### Next steps
- Harmonize the Excel template with project BIM Execution Plan requirements.
- Create a Dynamo player UI wrapper for quicker deployment to production teams.

## R23_GEN_QC_All_Incorrect_Levels_Up.dyn
- **Discipline**: General – quality control.
- **Purpose**: Identifies elements assigned to incorrect base levels and rehosts them to the closest level above based on user-provided rules.

### Inputs
- **Category list**: Multi-select input of categories to inspect (walls, floors, columns, ducts).
- **Tolerance**: Number specifying acceptable deviation in project units.
- **Level priority table**: Excel path that maps current levels to target levels.
- **Record-only toggle**: Boolean controlling whether to move elements or just report them.

### Outputs
- **Updated elements** returned as a list when rehosting occurs.
- **QA report** detailing element IDs, old level, new level, and delta.
- **Skipped elements list** for cases exceeding tolerance or locked constraints.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- BimorphNodes — version 5.2.2
- archi-lab.net — version 2023.213.1223

### Python nodes
1. **Level analyzer** – Calculates vertical deltas and selects the next highest level from the priority table.
2. **Rehost engine** – Moves elements via the Revit API, handling different categories with specialized logic.

### Graph metrics
- 110 nodes
- 137 connectors
- 2 code block nodes
- 5 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Dry-run visualization** – Create temporary model lines at target elevations before committing moves.
2. **Undo package** – Store original locations in an external CSV to support rollbacks.
3. **Element filters** – Allow filtering by design option or workset to avoid moving in-progress elements.

### Next steps
- Test against linked models to ensure only host elements are rehosted.
- Develop Dynamo Player UI prompts for selecting the Excel priority file.

## R23_GEN_QC_Incorrect_Levels.dyn
- **Discipline**: General – quality control.
- **Purpose**: Flags elements whose base or top constraints do not align with level standards without modifying the model.

### Inputs
- **Category selection**: List of categories to inspect.
- **Tolerance**: Number slider representing acceptable level variance.
- **Level standard list**: Text list of allowed levels.

### Outputs
- **Element report** listing offenders with associated deltas.
- **View filter set** created to highlight incorrect elements in active views.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- Data-Shapes — version 2023.2.1

### Python nodes
1. **Level auditor** – Compares element constraints against the provided standard list.

### Graph metrics
- 84 nodes
- 101 connectors
- 1 code block node
- 3 watch nodes
- 1 IronPython node

### Opportunities for improvement
1. **Historical tracking** – Append results to a central log so QA trends can be reviewed.
2. **Auto-filters** – Push generated filters into view templates automatically.
3. **Category presets** – Provide discipline-based presets (structural, architectural, MEP).

### Next steps
- Align tolerance defaults with project BIM Execution Plans.
- Bundle with the "All Incorrect Levels Up" script as a pre-check.

## R23_GEN_ResetOverrides.dyn
- **Discipline**: General – view management.
- **Purpose**: Resets temporary graphic overrides, filters, and element hides for selected views.

### Inputs
- **View selection**: `Select Model Elements` limited to views.
- **Reset filters toggle**: Boolean controlling whether view filters are removed or just disabled.
- **Template reapply toggle**: Boolean instructing the graph to reapply assigned view templates after clearing overrides.

### Outputs
- **Cleaned views** returned so users can review them downstream.
- **Action log** listing which overrides were reset per view.

### Packages
- Clockwork for Dynamo 2.x — version 2.12.3
- Genius Loci — version 2023.2.21
- Rhythm — version 2023.1.1

### Python nodes
1. **Override resetter** – Calls `View.SetElementOverrides`, `View.SetFilterOverrides`, and `View.GetFilters` to restore defaults.

### Graph metrics
- 62 nodes
- 74 connectors
- 1 code block node
- 2 watch nodes
- 1 IronPython node

### Opportunities for improvement
1. **Selective reset** – Allow users to target specific override categories (line color, patterns, etc.).
2. **Template comparison** – Report differences between the cleaned view and its template before reapplying.
3. **Batch logging** – Export the action log to CSV for audit trails.

### Next steps
- Test with dependent views to ensure overrides propagate correctly.
- Surface warnings when filters are removed so designers can reapply intentionally custom ones.

## R23_PLM_PI_PipesByCAD.dyn
- **Discipline**: Plumbing – pipe routing.
- **Purpose**: Converts CAD polylines into plumbing pipes with correct system types, slopes, and elevations.

### Inputs
- **CAD link**: `Select Model Element` referencing the plumbing DWG.
- **Layer/system map**: Excel-driven dictionary pairing CAD layers with Revit system and pipe types.
- **Nominal diameter**: Number input used when the layer map lacks explicit size data.
- **Slope value**: Numeric input representing rise over run.
- **Start elevation**: Number controlling vertical placement relative to host level.

### Outputs
- **Pipe elements** created through `MEPover.Pipe.ByCurve`.
- **System assignment** updates for `System Classification` and `System Type`.
- **Run summary** capturing length per system.

### Packages
- MEPover — version 1.6.0
- Clockwork for Dynamo 2.x — version 2.12.3
- BimorphNodes — version 5.2.2
- Data-Shapes — version 2023.2.1

### Python nodes
1. **Polyline collector** – Extracts CAD curves and ensures they are planar before conversion.
2. **System dispatcher** – Matches curves to system types based on the user map.
3. **Slope applier** – Adjusts Z coordinates along each pipe to match the requested slope.

### Graph metrics
- 196 nodes
- 231 connectors
- 4 code block nodes
- 8 watch nodes
- 3 IronPython nodes

### Opportunities for improvement
1. **Crossing resolution** – Detect intersections and prompt for vertical offsets automatically.
2. **Error visualization** – Create temporary lines where conversion failed for quick review.
3. **Performance** – Replace sequential list processing with chunked operations for large DWGs.

### Next steps
- Share standardized Excel templates for mechanical/plumbing coordination.
- Validate slope handling against sloped piping connectors in Revit 2025.

## R23_PLM_PI_Pipes_Fittings_ByCAD.dyn
- **Discipline**: Plumbing – fittings automation.
- **Purpose**: Inserts pipe fittings along converted runs by analyzing direction changes and tee branches from CAD geometry.

### Inputs
- **Pipe elements**: Output list from the `PipesByCAD` workflow.
- **Fitting family**: Drop-down referencing tee/elbow families for plumbing systems.
- **Branch layer filter**: String input listing CAD layers that should receive branch fittings.
- **Bend threshold**: Angle that triggers elbow placement.
- **Size override**: Number to enforce uniform fitting size when needed.

### Outputs
- **Fitting instances** placed via a Python helper that calls `MechanicalUtils.InsertFitting`.
- **Skipped connections** list for branch locations that lacked sufficient space.
- **Inventory table** summarizing fittings by type and size.

### Packages
- MEPover — version 1.6.0
- Clockwork for Dynamo 2.x — version 2.12.3
- Genius Loci — version 2023.2.21

### Python nodes
1. **Branch detector** – Identifies intersection points and classifies them as tee or cross connections.
2. **Placement routine** – Inserts fittings and orients connectors, handling size overrides where required.

### Graph metrics
- 138 nodes
- 165 connectors
- 2 code block nodes
- 6 watch nodes
- 2 IronPython nodes

### Opportunities for improvement
1. **Fabrication mapping** – Map fittings to fabrication part numbers for downstream workflows.
2. **Host verification** – Check that host pipes remain in the expected system before inserting fittings.
3. **Visualization** – Drop temporary markers at skipped intersections for manual follow-up.

### Next steps
- Coordinate with fabrication teams to validate size overrides and fitting families.
- Bundle this workflow with the pipe creation script inside Dynamo Player for field users.

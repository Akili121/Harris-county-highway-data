# Harris County Highway Network and Flood-Impact Data

This repository provides the data used in the Harris County case study presented in:

> Beixuan Dong, Xinming Li, and Lingzi Wu. **A GNN-Based Framework for Assessing Flood Impacts on Highway Networks: Integrating Network Structural, Functional, and Social Features.** *Journal of Management in Engineering*, 42(4), 04026010, 2026. [https://doi.org/10.1061/JMENEA.MEENG-7198](https://doi.org/10.1061/JMENEA.MEENG-7198)

The dataset combines a highway-network representation with time-varying traffic conditions, flood-related incidents and road closures, and census-tract-level social vulnerability information. It supports research on disaster impact assessment, transportation-network modeling, graph-based machine learning, and socially informed infrastructure resilience.

## Dataset at a glance

- **Study area:** Harris County, Texas, USA
- **Network:** 546 nodes and 557 directed edge records
- **Traffic period:** August 18, 2017, 00:00 to September 7, 2017, 23:45
- **Temporal resolution:** 15 minutes
- **Traffic observations:** 2,016 time steps for each of 557 edges
- **Flood-related records:** 474 high-water incidents and 39 road closures
- **Social vulnerability:** CDC Social Vulnerability Index values for 2016 and 2018

## Repository contents

| File | Records | Description |
| --- | ---: | --- |
| `node_file.csv` | 546 | Highway-network nodes with node IDs, latitude, and longitude. |
| `edge_file.csv` | 557 | Network edges with edge IDs, start/end node IDs, endpoint coordinates, and length in miles. |
| `Flow on the edges.csv` | 2,016 | Time-indexed traffic-flow observations. Columns `1`-`557` correspond to the edge IDs in `edge_file.csv`. |
| `Speed on the edges.csv` | 2,016 | Time-indexed traffic-speed observations. Columns `1`-`557` correspond to the edge IDs in `edge_file.csv`. |
| `highwater_incidents.csv` | 474 | High-water incident records, including roadway, location, weather, verification, response, lane impact, and event timing attributes. |
| `road_closure.csv` | 39 | Flood-related road-closure records with location, direction, blocked lanes, closure period, and description. |
| `nodes_with_svi_2016_2018.csv` | 550 | Node locations joined with census tract, FIPS code, and 2016/2018 Social Vulnerability Index values. |

## Data relationships

The files can be linked using the following identifiers:

```text
node_file.csv
    Node ID
       |
       +---- edge_file.csv (Start ID, End ID)
       |          |
       |          +---- Flow on the edges.csv (columns 1-557 = Edge ID)
       |          +---- Speed on the edges.csv (columns 1-557 = Edge ID)
       |
       +---- nodes_with_svi_2016_2018.csv (Node ID)
```

Incident and closure records contain geographic coordinates and roadway descriptors that can be spatially related to the network.

## Quick start

The following Python example loads the network and converts the wide traffic tables into long format for analysis.

```python
import pandas as pd

nodes = pd.read_csv("node_file.csv")
edges = pd.read_csv("edge_file.csv")
flow = pd.read_csv("Flow on the edges.csv", parse_dates=["Date Time"])
speed = pd.read_csv("Speed on the edges.csv", parse_dates=["Date Time"])
svi = pd.read_csv("nodes_with_svi_2016_2018.csv")

# Convert edge columns into one row per timestamp-edge observation.
flow_long = flow.melt(
    id_vars="Date Time",
    var_name="Edge ID",
    value_name="Flow",
)
speed_long = speed.melt(
    id_vars="Date Time",
    var_name="Edge ID",
    value_name="Speed",
)

flow_long["Edge ID"] = flow_long["Edge ID"].astype(int)
speed_long["Edge ID"] = speed_long["Edge ID"].astype(int)

traffic = flow_long.merge(speed_long, on=["Date Time", "Edge ID"])
traffic = traffic.merge(edges, on="Edge ID", validate="many_to_one")

print(nodes.shape)    # (546, 3)
print(edges.shape)    # (557, 6)
print(traffic.head())
```

## Notes and data-quality considerations

- The traffic matrices use edge IDs as column names; these IDs correspond directly to `Edge ID` in `edge_file.csv`.
- `nodes_with_svi_2016_2018.csv` contains 550 rows. Four rows have geographic and SVI information but no `Node ID`; users joining by node ID should review or exclude these rows as appropriate for their analysis.
- Missing values, event durations, coordinate fields, and categorical encodings should be reviewed before modeling.
- Consult the associated paper for the study design, preprocessing decisions, feature construction, and modeling methodology.
- This repository currently does not include a license file. Please contact the repository owner before redistribution or uses beyond research and citation of the associated study.

## Citation

If you use this dataset, please cite the associated paper:

```bibtex
@article{dong2026gnn,
  title   = {A GNN-Based Framework for Assessing Flood Impacts on Highway Networks: Integrating Network Structural, Functional, and Social Features},
  author  = {Dong, Beixuan and Li, Xinming and Wu, Lingzi},
  journal = {Journal of Management in Engineering},
  volume  = {42},
  number  = {4},
  pages   = {04026010},
  year    = {2026},
  doi     = {10.1061/JMENEA.MEENG-7198}
}
```

## Contact

For questions about the dataset, please contact **Beixuan Dong** at [beixuan1@ualberta.ca](mailto:beixuan1@ualberta.ca).

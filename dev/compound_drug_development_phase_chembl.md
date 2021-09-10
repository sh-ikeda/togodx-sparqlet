# ChEMBLを薬の開発フェーズで分類する（信定） 作業中


## Description

- Data sources
    - (More data sources description goes here..)
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
    - Output
        - Highest development phase
        
 ## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT?parent ?child ?child_label  
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>

WHERE 
{
 ?drug cco:chemblId ?child ;
            rdfs:label ?child_label ;
            cco:highestDevelopmentPhase ?parent .
  filter not exists { ?drug a cco:DrugIndication }
}
limit 1000
```
## `return`

```javascript
({data}) => {
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let edge = {};
  data.results.bindings.map(d => {
  
    // development_phase にラベルをつける
    let parent_label = d.parent.value;
    if (parent_label  == 0) parent_label = "0: No description";
    else if (parent_label  == 1) parent_label = "1: PK tolerability";
    else if (parent_label  == 2) parent_label = "2: Efficacy";
    else if (parent_label  == 3) parent_label = "3: Safety & Efficacy";
    else  (parent_label  == 4) parent_label = "4: Indication Discovery & expansion";
    
    tree.push({
      id: d.child.value,
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value
    })
  // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({   
        id: d.parent.value,
        label: parent_label,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```
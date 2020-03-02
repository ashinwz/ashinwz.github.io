##Spotfire IronPython Usage

<mark>Usual Usage of IronPython</mark>

###Visualization
```python
from Spotfire.Dxp.Data import *

#get the data table
one_table = Document.Data.Tables["target table"]
#place generic data cursor on a specific column
cursor = DataValueCursor.CreateFormatted(one_table.Columns["colID"])
# Retrieve the marking selection
markings = Document.ActiveMarkingSelectionReference.GetSelection(one_table)
# Iterate through the data table rows to retrieve the marked rows
mark = {}
for row in one_table.GetRows(markings.AsIndexSet(),cursor):
    value = cursor.CurrentValue
    mark[value] = 1

```


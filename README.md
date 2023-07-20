# How-to-move-the-next-cell-into-edit-mode-by-pressing-the-tab-key-in-Flutter-DataTable-SfDataGrid-

The [Flutter DataGrid](https://www.syncfusion.com/flutter-widgets/flutter-datagrid) provides support for moving a cell into edit mode programmatically by calling the [DataGridController.beginEdit](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/DataGridController/beginEdit.html) method. In this article, you can learn about how to move the next cell into edit mode by pressing the tab key on the keyboard.

## STEP 1: 
Create a custom selection manager class by extending the [RowSelectionManager](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/RowSelectionManager-class.html). In the [handleKeyEvent](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/RowSelectionManager/handleKeyEvent.html) method, call the `DataGridController.beginEdit` inside the WidgetsBinding.instance.addPostFrameCallback using the [DataGridController.currentCell](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/DataGridController/currentCell.html).

```dart
class CustomSelectionManager extends RowSelectionManager {
  CustomSelectionManager(this.dataGridController);
  DataGridController dataGridController;
  @override
  Future<void> handleKeyEvent(RawKeyEvent keyEvent) async {
    if (keyEvent.logicalKey == LogicalKeyboardKey.tab) {
      WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
        dataGridController.beginEdit(dataGridController.currentCell);
      });
    }
    super.handleKeyEvent(keyEvent);
  }
}

```
## STEP 2: 
Create a data source class by extending the [DataGridSource](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/DataGridSource-class.html) for mapping data to the SfDataGrid and add the required edit widget through the [DataGridSource.buildEditWidget](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/DataGridSource/buildEditWidget.html) method in the DataGridSource class.

```dart
class EmployeeDataSource extends DataGridSource {
  EmployeeDataSource(this.employees, this.buildContext) {
    dataGridRows = employees
        .map<DataGridRow>((dataGridRow) => dataGridRow.getDataGridRow())
        .toList();
  }

  List<Employee> employees = [];

  List<DataGridRow> dataGridRows = [];

  BuildContext buildContext;

  /// Helps to hold the new value of all editable widget.
  /// Based on the new value we will commit the new value into the corresponding
  /// [DataGridCell] on [onSubmitCell] method.
  dynamic newCellValue;

  /// Help to control the editable text in [TextField] widget.
  TextEditingController editingController = TextEditingController();

  @override
  List<DataGridRow> get rows => dataGridRows;

  @override
  DataGridRowAdapter? buildRow(DataGridRow row) {
    return DataGridRowAdapter(
        cells: row.getCells().map<Widget>((dataGridCell) {
      return Container(
          alignment: (dataGridCell.columnName == 'id' ||
                  dataGridCell.columnName == 'salary')
              ? Alignment.centerRight
              : Alignment.centerLeft,
          padding: const EdgeInsets.symmetric(horizontal: 8.0),
          child: Text(
            dataGridCell.value.toString(),
            overflow: TextOverflow.ellipsis,
          ));
    }).toList());
  }

  @override
  Future<void> onCellSubmit(DataGridRow dataGridRow, RowColumnIndex rowColumnIndex,
      GridColumn column) async {
    final dynamic oldValue = dataGridRow
            .getCells()
            .firstWhereOrNull((DataGridCell dataGridCell) =>
                dataGridCell.columnName == column.columnName)
            ?.value ??
        '';

    final int dataRowIndex = dataGridRows.indexOf(dataGridRow);

    if (newCellValue == null || oldValue == newCellValue) {
      return;
    }

    if (column.columnName == 'id') {
      dataGridRows[dataRowIndex].getCells()[rowColumnIndex.columnIndex] =
          DataGridCell<int>(columnName: 'id', value: newCellValue);
      employees[dataRowIndex].id = newCellValue as int;
    } else if (column.columnName == 'name') {
      dataGridRows[dataRowIndex].getCells()[rowColumnIndex.columnIndex] =
          DataGridCell<String>(columnName: 'name', value: newCellValue);
      employees[dataRowIndex].name = newCellValue.toString();
    } else if (column.columnName == 'designation') {
      dataGridRows[dataRowIndex].getCells()[rowColumnIndex.columnIndex] =
          DataGridCell<String>(columnName: 'designation', value: newCellValue);
      employees[dataRowIndex].designation = newCellValue.toString();
    } else {
      dataGridRows[dataRowIndex].getCells()[rowColumnIndex.columnIndex] =
          DataGridCell<int>(columnName: 'salary', value: newCellValue);
      employees[dataRowIndex].salary = newCellValue as int;
    }
  }

  @override
  Widget? buildEditWidget(DataGridRow dataGridRow,
      RowColumnIndex rowColumnIndex, GridColumn column, CellSubmit submitCell) {
    // Text going to display on editable widget
    final String displayText = dataGridRow
            .getCells()
            .firstWhereOrNull((DataGridCell dataGridCell) =>
                dataGridCell.columnName == column.columnName)
            ?.value
            ?.toString() ??
        '';

    // The new cell value must be reset.
    // To avoid committing the [DataGridCell] value that was previously edited
    // into the current non-modified [DataGridCell].
    newCellValue = null;

    final bool isNumericType =
        column.columnName == 'id' || column.columnName == 'salary';

    // Holds regular expression pattern based on the column type.
    final RegExp regExp = _getRegExp(isNumericType, column.columnName);

    return Container(
      padding: const EdgeInsets.all(8.0),
      alignment: isNumericType ? Alignment.centerRight : Alignment.centerLeft,
      child: TextField(
        controller: editingController..text = displayText,
        textAlign: isNumericType ? TextAlign.right : TextAlign.left,
        autofocus: true,
        decoration: const InputDecoration(
          contentPadding: EdgeInsets.fromLTRB(0, 0, 0, 8.0),
        ),
        inputFormatters: <TextInputFormatter>[
          FilteringTextInputFormatter.allow(regExp)
        ],
        keyboardType: isNumericType ? TextInputType.number : TextInputType.text,
        onChanged: (String value) {
          if (value.isNotEmpty) {
            if (isNumericType) {
              newCellValue = int.parse(value);
            } else {
              newCellValue = value;
            }
          } else {
            newCellValue = null;
          }
        },
        onSubmitted: (String value) {
          /// Call [CellSubmit] callback to fire the canSubmitCell and
          /// onCellSubmit to commit the new value in single place.
          submitCell();
        },
      ),
    );
  }

  RegExp _getRegExp(bool isNumericKeyBoard, String columnName) {
    return isNumericKeyBoard ? RegExp('[0-9]') : RegExp('[a-zA-Z ]');
  }
}

```
## STEP 3: 
Initialize the [SfDataGrid](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/SfDataGrid-class.html) widget with all the required properties. Create an instance of the CustomSelectionManager class and assign it to the [SfDataGrid.selectionManager](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/SfDataGrid/selectionManager.html) property. Create an instance of the DataGridController and assign it to the [SfDataGrid.controller](https://pub.dev/documentation/syncfusion_flutter_datagrid/latest/datagrid/SfDataGrid/controller.html) property.

```dart
List<Employee> _employees = <Employee>[];
  late EmployeeDataSource _employeeDataSource;
  DataGridController dataGridController = DataGridController();
  late CustomSelectionManager customSelectionManager;

  @override
  void initState() {
    super.initState();
    _employees = getEmployeeData();
    _employeeDataSource = EmployeeDataSource(_employees, context);
    customSelectionManager = CustomSelectionManager(dataGridController);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Syncfusion Flutter DataGrid')),
      body: SfDataGrid(
          source: _employeeDataSource,
          controller: dataGridController,
          allowEditing: true,
          selectionManager: customSelectionManager,
          navigationMode: GridNavigationMode.cell,
          selectionMode: SelectionMode.single,
          columns: <GridColumn>[
            GridColumn(
                columnName: 'id',
                label: Container(
                    padding: const EdgeInsets.all(8.0),
                    alignment: Alignment.centerRight,
                    child: const Text('ID'))),
            GridColumn(
                columnName: 'name',
                label: Container(
                    padding: const EdgeInsets.all(8.0),
                    alignment: Alignment.centerLeft,
                    child: const Text('Name'))),
            GridColumn(
                columnName: 'designation',
                label: Container(
                    padding: const EdgeInsets.all(8.0),
                    alignment: Alignment.centerLeft,
                    child: const Text('Designation',
                        overflow: TextOverflow.ellipsis))),
            GridColumn(
                columnName: 'salary',
                label: Container(
                    padding: const EdgeInsets.all(8.0),
                    alignment: Alignment.centerRight,
                    child: const Text('Salary'))),
          ],
          columnWidthMode: ColumnWidthMode.fill),
    );
  }

```
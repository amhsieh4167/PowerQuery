(previousStep as table, columnsToUnpivot as list, optional arg_numberOfColumnsToSplit as number) as table =>   
let    
    numberOfColumnsToSplit = arg_numberOfColumnsToSplit ?? 2,
    #"Added Index" = Table.AddIndexColumn(previousStep, "Index", 0, 1, Int64.Type),
    SplitColumns =  Table.Buffer(
                        Table.Combine(
                            List.Transform(
                                List.Split(
                                    columnsToUnpivot,
                                    numberOfColumnsToSplit
                                ),
                                each Table.RenameColumns(
                                        Table.SelectColumns(
                                            #"Added Index", {"Index"} & _
                                        ),
                                        List.Zip({_, {"Charge Type", "Service Amount"}})
                                     )
                            )
                        )
                    ),
    #"Filtered out null and 0" = Table.SelectRows(SplitColumns, each ([Service Amount] <> null and [Service Amount] <> 0)),
    UniqueChargeType = List.Buffer(List.Distinct(#"Filtered out null and 0"[Charge Type])),
    #"Pivoted Column" = Table.Pivot(#"Filtered out null and 0", UniqueChargeType, "Charge Type", "Service Amount"),
    RemovePivotedColumns = Table.RemoveColumns(#"Added Index", columnsToUnpivot),
    MergeBacktoMain = Table.NestedJoin(
                        RemovePivotedColumns, {"Index"},
                        #"Pivoted Column", {"Index"},
                        "Data"
                      ),
    ExpandedData = Table.ExpandTableColumn(MergeBacktoMain, "Data", UniqueChargeType),
    ReplacedNull = Table.ReplaceValue(ExpandedData, null, 0, Replacer.ReplaceValue, UniqueChargeType),
    RemoveIndex = Table.RemoveColumns(ReplacedNull, {"Index"})
in    
    RemoveIndex

const filterColumns = useMemo((): TableProps<T>["columns"] => {
      const res = columns.filter((item) =>
        checkColumns.includes(item.key as string)
      );
      if (curSimulation) {
        res[1].filters = curSimulation.codes.map((codeInfo) => ({
          text: codeInfo.code,
          value: codeInfo.code,
        }));
        res[1].filterMultiple = true;
        res[1].filterSearch = true;
        res[1].onFilter = (val, record) =>
          record.code.toLowerCase() === (val as string).toLowerCase();
        res[1].filteredValue = [...selectedCodes];
      }
      return res;
    }, [
      checkColumns,
      curSimulation?.codes, // 注意：只依赖 codes，而不是整个 curSimulation 对象
      selectedCodes, // filteredValue 的来源
    ]);

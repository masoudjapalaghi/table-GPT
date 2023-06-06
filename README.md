import React, { useState, useEffect } from 'react';

interface Column {
  key: string;
  label: string;
}

interface TableProps {
  url: string;
  columns: Column[];
  pageSize?: number;
}

const useSort = (data: any[], defaultSortColumn: string | null = null, defaultSortDirection: string | null = null) => {
  const [sortColumn, setSortColumn] = useState<string | null>(defaultSortColumn);
  const [sortDirection, setSortDirection] = useState<string | null>(defaultSortDirection);

  const handleSort = (column: string) => {
    if (sortColumn === column) {
      setSortDirection(sortDirection === 'asc' ? 'desc' : 'asc');
    } else {
      setSortColumn(column);
      setSortDirection('asc');
    }
  };

  const sortedData = sortColumn ? [...data].sort((a, b) => {
    if (sortDirection === 'asc') {
      return a[sortColumn] < b[sortColumn] ? -1 : 1;
    } else {
      return a[sortColumn] > b[sortColumn] ? -1 : 1;
    }
  }) : data;

  return [sortedData, sortColumn, sortDirection, handleSort] as const;
};

const usePagination = (data: any[], pageSize: number = 10, defaultPage: number = 1) => {
  const [currentPage, setCurrentPage] = useState<number>(defaultPage);

  const pageCount = Math.ceil(data.length / pageSize);
  const pageData = data.slice((currentPage - 1) * pageSize, currentPage * pageSize);

  const handlePageChange = (page: number) => {
    setCurrentPage(page);
  };

  return [pageData, pageCount, currentPage, handlePageChange] as const;
};

const Table: React.FC<TableProps> = ({ url, columns, pageSize = 10 }) => {
  const [data, setData] = useState<any[]>([]);

  useEffect(() => {
    const fetchData = async () => {
      const response = await fetch(url);
      const json = await response.json();
      setData(json);
    };
    fetchData();
  }, [url]);

  const [sortedData, sortColumn, sortDirection, handleSort] = useSort(data);
  const [pageData, pageCount, currentPage, handlePageChange] = usePagination(sortedData, pageSize);

  useEffect(() => {
    const searchParams = new URLSearchParams();
    if (sortColumn) {
      searchParams.set('sortColumn', sortColumn);
    }
    if (sortDirection) {
      searchParams.set('sortDirection', sortDirection);
    }
    if (currentPage !== 1) {
      searchParams.set('page', currentPage.toString());
    }
    const newUrl = `${window.location.pathname}?${searchParams.toString()}`;
    window.history.pushState({}, '', newUrl);
  }, [sortColumn, sortDirection, currentPage]);

  return (
    <div>
      <table>
        <thead>
          <tr>
            {columns.map((column) => (
              <th key={column.key} onClick={() => handleSort(column.key)}>
                {column.label}
                {sortColumn === column.key && (
                  sortDirection === 'asc' ? ' ▲' : ' ▼'
                )}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {pageData.map((row, index) => (
            <tr key={index}>
              {columns.map((column) => (
                <td key={column.key}>
                  {row[column.key]}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
      <div>
        {Array.from({ length: pageCount }, (_, i) => i + 1).map((page) => (
          <button key={page} onClick={() => handlePageChange(page)}>
            {page}
          </button>
        ))}
      </div>
    </div>
  );
};

export default Table;

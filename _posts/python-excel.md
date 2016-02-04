---
layout: page
title:
category: blog
description:
---
# Preface


# openpyxl

	from openpyxl import load_workbook
	wb = load_workbook(filename = 'empty_book.xlsx')
	for sheetname in wb.sheetnames:
		ws = wb[sheetname]
	for ws in wb.worksheets:
		print(ws['A1'].value)

## row, cell

	from openpyxl import load_workbook
	wb = load_workbook(filename='large_file.xlsx', read_only=True)
	ws = wb['big_data'] # ws is now an IterableWorksheet

	for row in ws.rows:
		for cell in row:
			print(cell.value)

## csv

	import openpyxl
	import csv

	wb = openpyxl.load_workbook('test.xlsx')
	print(wb.sheetnames)
	sh = wb.active
	# python3 only
	with open('test.csv', 'w') as f:
		c = csv.writer(f)
		for r in sh.rows:
			c.writerow([cell.value for cell in r])

.PHONY: clean pdf

MAIN=main

all:
	latexmk -pdf $(MAIN)

clean:
	latexmk -c $(MAIN)

pdf:
	pdflatex $(MAIN)
	bibtex $(MAIN)
	pdflatex $(MAIN)
	pdflatex $(MAIN)

#clean:
#	rm *.log *.aux *.bbl *.blg *~ *.fdb_latexmk *.fls *.out *.gz

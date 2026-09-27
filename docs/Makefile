# Turk-OS study plan -- build every document into pdf/ (aux files go to build/).
#   make                          build the roadmap and all phase documents
#   make pdf/phase-03-gdt-idt-exceptions.pdf   build a single document
#   make clean | distclean
LATEXMK := latexmk -pdf -interaction=nonstopmode -halt-on-error -outdir=pdf -auxdir=build

SOURCES := 00-roadmap.tex $(sort $(wildcard phases/*.tex))
PDFS    := $(patsubst %.tex,pdf/%.pdf,$(notdir $(SOURCES)))

.PHONY: all clean distclean
all: $(PDFS)

pdf/00-roadmap.pdf: 00-roadmap.tex common/turkos.sty
	$(LATEXMK) $<

pdf/%.pdf: phases/%.tex common/turkos.sty
	$(LATEXMK) $<

clean:
	rm -rf build

distclean: clean
	rm -rf pdf

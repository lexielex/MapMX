# MapMX
Shapefiles, datos y herramientas para mapas de México

# Makefile
GENERATED_FILES = \
	mx.json

.PHONY: all clean

all: $(GENERATED_FILES)

clean:
	rm -rf -- $(GENERATED_FILES) build

build/mgm2010v5_0a.zip:
	mkdir -p $(dir $@)
	curl -o $@ 'http://mapserver.inegi.org.mx/MGN/mgm2010v5_0a.zip'

build/mgm2010v5_0a.shp: build/mgm2010v5_0a.zip
	rm -rf -- $(basename $@)
	mkdir -p $(basename $@)
	unzip -d $(basename $@) $<
	for file in `find $(basename $@) -type f`; do chmod 644 $$file; mv $$file $(basename $@).$${file##*.}; done
	rm -rf -- $(basename $@)
	touch $@

mx.json: build/mgm2010v5_0a.shp
	node_modules/.bin/geo2topo --width=960 --margin=20 --simplify=1 -q 1e5 -o $@ -p NOM_MUN=NOM_MUN -p CVE_ENT=+CVE_ENT -p CVE_MUN=+CVE_MUN -- municipalities=$<

### Problemas con topojson/geo2topo
Hay issues con instalación de topojson/geo2topo que hacen que el Makefile no jale en algunos casos. 
El topojson de municipios (mx.json) ya tiene nombre, y códigos de estado y municipio.

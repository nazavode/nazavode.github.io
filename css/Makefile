stylesheets := normalize.css style.css

all.min.css : $(stylesheets)
	cat $(stylesheets) | curl -X POST -s --data-urlencode input@- https://cssminifier.com/raw > $@

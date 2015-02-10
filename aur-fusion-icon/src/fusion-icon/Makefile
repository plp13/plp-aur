PREFIX = '/usr'
DESTDIR = '/'
#interfaces = 'gtk qt4'

all:
	@python2 setup.py build

install:
	@python2 setup.py install --prefix=${PREFIX} --root=${DESTDIR}

uninstall:
	@python2 setup.py uninstall

clean:
	rm -rf build/


# Preliminaries
import os, re
#import SCons.SCons.Sconsign

# Test for proper prerequisites and setup
from setup import setup_test
[user_configs, mode, sf, cache_dir] = setup_test(ARGUMENTS)
import gslab_scons 
import yaml
import atexit
import gslab_scons.log as log

# Start log
mode = ARGUMENTS.get('mode', 'develop') # Gets mode; defaults to 'develop'
vers = ARGUMENTS.get('version', '') # Gets release version; defaults to ''
log.start_log(mode, vers)

# Defines environment
env = Environment(ENV = {'PATH' : os.environ['PATH']}, 
                  IMPLICIT_COMMAND_DEPENDENCIES = 0,
                  BUILDERS = {'Tablefill'   : Builder(action = gslab_scons.build_tables),
                              'BuildLyx'    : Builder(action = gslab_scons.build_lyx),
                              'BuildR'      : Builder(action = gslab_scons.build_r),
                              'BuildStata'  : Builder(action = gslab_scons.build_stata),
                              'BuildPython' : Builder(action = gslab_scons.build_python)},
                  user_flavor = sf)


def parse_sconsign_for_info(tgt):
	sconsign = os.popen('sconsign .sconsign.dblite').readlines() # http://stackoverflow.com/questions/3503879/assign-output-of-os-system-to-a-variable-and-prevent-it-from-being-displayed-on
	tgt_dir = '/'.join(tgt.split('/')[0:(len(tgt.split('/')) - 1)])
	tgt_file = tgt.split('/')[len(tgt.split('/')) - 1]
	dir_flag = 0
	for line in sconsign:
		if dir_flag == 1:
			if re.search('^' + tgt_file, line):
				csig = line.split(' ')[1].strip()
				time_stamp = line.split(' ')[2].strip()
				size = line.split(' ')[3].strip()
				return csig, time_stamp, size
		else:
			if re.search(tgt_dir, line):
				dir_flag = 1

def new_decider(dependency, target, prev_ni):
	tgt = str(target)
	tgt_csig, tgt_time_stamp, tgt_size = parse_sconsign_for_info(tgt)
	target_info = os.stat(str(target))

	# Check target time-stamp and then content signature
	if float(tgt_time_stamp) != target_info.st_mtime:
		if tgt_csig != target.get_csig():
			return True
	# Check source files
	else:
		return dependency.changed_timestamp_then_content(target, prev_ni)


env.Decider('MD5-timestamp') # Only computes hash if time-stamp changed
env.EXTENSIONS = ['.eps', '.pdf', '.lyx'] # Extensions to be used when scanning for source files in BuildLyx.
SourceFileScanner.add_scanner('.lyx', Scanner(gslab_scons.misc.lyx_scan, recursive = True))

# Load paths
env['PATHS'] = yaml.load(open("constants.yaml", 'rU'))

# Export environment
Export('env')

# Run sub-trees
SConscript('source/data/SConscript') 
SConscript('source/analysis/SConscript')
SConscript('source/tables/SConscript') 
SConscript('source/paper/SConscript') 
SConscript('source/talk/SConscript') 
Default('./build', './release')

# Additional mode options
if mode == 'cache':
    CacheDir(cache_dir)

# Print the state of the repo at end of SCons run
finish_command = Command( 'state_of_repo.log', [], gslab_scons.misc.state_of_repo, MAXIT=10) # From http://stackoverflow.com/questions/8901296/how-do-i-run-some-code-after-every-build-in-scons
Depends(finish_command, BUILD_TARGETS)
env.AlwaysBuild(finish_command)
if 'state_of_repo.log' not in BUILD_TARGETS: 
    BUILD_TARGETS.append('state_of_repo.log')

atexit.register(log.end_log)


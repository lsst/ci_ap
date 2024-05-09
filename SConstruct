# -*- python -*-
import os
from SCons.Script import Environment, GetOption, Default
from lsst.sconsUtils.utils import libraryLoaderEnvironment
import lsst.utils.threads

env = Environment(ENV=os.environ)
lsst.utils.threads.disable_implicit_threading()


def getExecutableCmd(package, script, *args, directory="bin"):
    """
    Given the name of a package and a script or other executable which lies
    within the given subdirectory, return an appropriate string which can be
    used to set up an appropriate environment and execute the command.
    This includes:
    * Specifying an explicit list of paths to be searched by the dynamic
      linker;
    * Specifying a Python executable to be run (we assume the one on the
      default ${PATH} is appropriate);
    * Specifying the complete path to the script.
    """
    cmds = [libraryLoaderEnvironment(), "python",
            os.path.join(env.ProductDir(package), directory, script)]
    cmds.extend(args)
    # NOTE: if `directory` or any of the `args` contain spaces, this will
    # fail because we are not doing shell quoting here.
    return " ".join(cmds)


# TODO: do we need any python here?
# scripts.BasicSConstruct("ci_ap", disableCc=True, noCfgFile=True)

num_process = GetOption('num_jobs')

PKG_ROOT = env.ProductDir("ci_ap")
DC2_ROOT = os.path.join(PKG_ROOT, "output-dc2")
COSMOS_PDR2_ROOT = os.path.join(PKG_ROOT, "output-cosmos_pdr2")
HITS2015_ROOT = os.path.join(PKG_ROOT, "output-hits2015")

dc2_pipeline = os.path.join(env.ProductDir("ap_verify_ci_dc2"), "pipelines", "ApVerifyWithFakes.yaml")
dc2 = env.Command(DC2_ROOT, None,
                  [getExecutableCmd("ap_verify", "ap_verify.py",
                                    "--dataset", "ap_verify_ci_dc2",
                                    "--pipeline", dc2_pipeline,
                                    "--output", DC2_ROOT,
                                    "-j", str(num_process))])

cosmos_pdr2 = env.Command(COSMOS_PDR2_ROOT, None,
                          [getExecutableCmd("ap_verify", "ap_verify.py",
                                            "--dataset", "ap_verify_ci_cosmos_pdr2",
                                            "--output", COSMOS_PDR2_ROOT,
                                            "-j", str(num_process))])

hits2015 = env.Command(HITS2015_ROOT, None,
                       [getExecutableCmd("ap_verify", "ap_verify.py",
                                         "--dataset", "ap_verify_ci_hits2015",
                                         "--output", HITS2015_ROOT,
                                         "-j", str(num_process))])

everything = [dc2, cosmos_pdr2, hits2015]
# Ensure that the above are not run in parallel (to avoid processor
# contention) by claiming that they all create a (non-existent) file
# "parallelization-lock".
env.SideEffect("parallelization-lock", everything)
env.Alias("all", everything)
env.Alias("dc2", dc2)
env.Alias("cosmos", cosmos_pdr2)
env.Alias("hits", hits2015)
Default(everything)

# Add a no-op install target to keep Jenkins happy.
env.Alias("install", "SConstruct")

# Double loop because "everything" contains all the dataset Commands, and each
# Command contains a list of its outputs.
env.Clean(everything, [y for x in everything for y in x])

# TODO: Do we have any tests we could run to check the output?
# state.env.Depends(state.targets["tests"], state.targets["data"])

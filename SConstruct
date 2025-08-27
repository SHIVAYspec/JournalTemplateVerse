import os
import json
import sys
from SCons.Script import Glob

env = Environment(tools=[])

build_dir = 'build'
tmpl_build_dir = os.path.join(build_dir,'tmpl')

contents_dir = 'contents'

def get_files(directory):
    files_list = list()
    for root, _, files in os.walk(directory):
        for file in files:
            files_list.append(os.path.join(root,file))
    return files_list

contents_files = get_files(contents_dir)

class Templates():
    def __init__(self,path):
        self.path = path
    
    def get_name(self):
        return os.path.basename(self.path)

    def get_compilers(self):
        return os.listdir(os.path.join(self.path, 'compilers'))

    def get_template_files(self):
        prefixlen = len(self.path) + 1
        return list(filter(lambda i: not i.startswith("compilers/",prefixlen), get_files(self.path)))

    def register_it(self):
        curr_tmpl_build_dir = os.path.join(tmpl_build_dir,self.get_name())
        # Copy the template files
        tmpldep = list()
        prefixlen = len(self.path) + 1
        for i in self.get_template_files():
            tmpldep.append(env.InstallAs(os.path.join(curr_tmpl_build_dir, i[prefixlen:]),i))
        # Preprocess and copy contents files
        contentsdep = list()
        compilers = set(self.get_compilers())
        # print(f"{self.get_name()} Compilers -> {compilers}", file=sys.stderr)
        prefixlen = len(contents_dir) + 1
        for i in contents_files:
            iextension = os.path.splitext(i)[1][1:]
            print(f"{self.get_name()} format {i} -> {iextension}", file=sys.stderr)
            if iextension in compilers:
                contentsdep.append(env.Command(
                    target=os.path.join(curr_tmpl_build_dir, i[prefixlen:-1 * len(iextension) - 1]),
                    source=i,
                    action=f"{self.path}/compilers/{iextension} $SOURCES > $TARGET"
                ))
            else:
                contentsdep.append(env.InstallAs(os.path.join(curr_tmpl_build_dir,i[prefixlen:]),i))
        target = os.path.join(curr_tmpl_build_dir, 'build', 'paper.pdf')
        source = os.path.join(curr_tmpl_build_dir,'paper.tex')
        output = env.Command(
                target = target,
                source = source,
                action = f"latexmk -cd -outdir=build -pdf $SOURCES"
        )
        for i in tmpldep + contentsdep:
            env.Depends(output, i)
        return output

    def __repr__(self):
        return ("*" * 100) + '\n' + json.dumps({
            'path' : self.path,
            'name' : self.get_name(),
            'compilers' : self.get_compilers(),
            'files' : self.get_template_files(),
        }, indent="\t")

templates_dir = 'templates'

templates_objs = list(map(lambda i: Templates(os.path.join(templates_dir,i)), os.listdir(templates_dir)))

# for i in templates_objs:
#     print(i,file=sys.stderr)

env.Default(env.Alias('all',[i.register_it() for i in templates_objs]))
# env.Command(target='mainfn', source=contents_files, action='echo DEBUG $SOURCE -> $TARGET;')

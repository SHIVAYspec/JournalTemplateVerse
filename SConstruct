import os
import json
import sys
import SCons.Script

env = Environment(tools=[])


def get_files(directory, skip=None):
    if skip:
        skip = os.path.join(directory,skip)
    files_list = list()
    for root, _, files in os.walk(directory):
        if skip and root.startswith(skip):
            continue
        for file in files:
            files_list.append(os.path.join(root,file))
    return files_list

contents_compilers = dict()
contents_compilers['dot'] = lambda target, source: env.Command(
        target=target,
        source=source,
        action=f"dot -T{os.path.splitext(target)[1][1:]} $SOURCE -o $TARGET"
    )

class Contents():
    def __init__(self,path):
        self.path = path
        self.files = get_files(path)

    def get_name(self):
        return os.path.basename(self.path)

    def get_path(self):
        return self.path

    def get_files(self):
        return self.files

    def __repr__(self):
        return json.dumps({
            "name": self.get_name(),
            "path": self.get_path(),
            "files": self.get_files()
        }, indent="\t")

class Templates():
    def __init__(self,path):
        self.path = path
        self.compilers = os.listdir(os.path.join(path, 'compilers'))
        self.files = get_files(self.path, 'compilers')
    
    def get_name(self):
        return os.path.basename(self.path)

    def get_path(self):
        return self.path

    def get_template_files(self):
        return self.files

    def get_compilers(self):
        return self.compilers

    def register_it(self,contents,build_dir):
        # Copy the template files
        tmpldep = list()
        prefixlen = len(self.path) + 1
        for i in self.get_template_files():
            tmpldep.append(env.InstallAs(os.path.join(build_dir, i[prefixlen:]),i))
        # Preprocess and copy contents files
        contentsdep = list()
        compilers = set(self.get_compilers())
        # print(f"{self.get_name()} Compilers -> {compilers}", file=sys.stderr)
        prefixlen = len(contents.get_path()) + 1
        for i in contents.get_files():
            iextension = os.path.splitext(i)[1][1:]
            # print(f"{self.get_name()} format {i} -> {iextension}", file=sys.stderr)
            if iextension in contents_compilers:
                contentsdep.append(
                    contents_compilers[iextension](
                        os.path.join(
                            build_dir,
                            i[prefixlen:-1 * len(iextension) - 1]
                        ),
                        i
                    )
                )
            elif iextension in compilers:
                compiler = f"{self.path}/compilers/{iextension}"
                temp_target = os.path.join(build_dir, i[prefixlen:-1 * len(iextension) - 1]),
                contentsdep.append(env.Command(
                    target=temp_target,
                    source=i,
                    action=f"{compiler} $SOURCES > $TARGET"
                ))
                env.Depends(temp_target,compiler)
            else:
                contentsdep.append(env.InstallAs(os.path.join(build_dir,i[prefixlen:]),i))
        target = os.path.join(build_dir, 'build', 'paper.pdf')
        source = os.path.join(build_dir,'paper.tex')
        output = env.Command(
                target = target,
                source = source,
                action = f"latexmk -cd -outdir=build -pdf $SOURCES"
        )
        for i in tmpldep + contentsdep:
            env.Depends(output, i)
        return output

    def __repr__(self):
        return json.dumps({
            'path' : self.path,
            'name' : self.get_name(),
            'compilers' : self.get_compilers(),
            'files' : self.get_template_files(),
        }, indent="\t")

contents_dir = 'contents'
icontents = SCons.Script.ARGUMENTS.get('contents', None)
if icontents:
    icontents = set(icontents.split(','))
else:
    icontents = os.listdir(contents_dir)
contents_list = [Contents(os.path.join(contents_dir,i)) for i in icontents]

for i in contents_list:
    print(f"Contents: {i.get_name()} : {'*' * (50 - len(i.get_name()))}")
    print(i,file=sys.stderr)

templates_dir = 'templates'
itemplates = SCons.Script.ARGUMENTS.get('templates', None)
if itemplates:
    itemplates = set(itemplates.split(','))
else:
    itemplates = os.listdir(templates_dir)
templates_list = [Templates(os.path.join(templates_dir,i)) for i in itemplates]

build_dir = 'build'

for i in templates_list:
    print(f"Template: {i.get_name()} : {'*' * (50 - len(i.get_name()))}")
    print(i,file=sys.stderr)

all = list()
for content in contents_list:
    for template in templates_list:
        all.append(
            template.register_it(
                content,os.path.join(
                    build_dir,
                    content.get_name(),
                    template.get_name()
                )
            )
        )

env.Default(env.Alias('all',all))

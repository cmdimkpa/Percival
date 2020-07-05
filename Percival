# Percival Interpreter
PercivalVersion =  0.5
# Monty Dimkpa

# -----------------------------------------------------------------
# Percival is a micro-programming language for performing ETL tasks
# -----------------------------------------------------------------

import subprocess

def run_shell(cmd):
  p = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
  out, err = p.communicate()
  if err:
      return err
  else:
      try:
          return eval(out)
      except:
          return out

# ensure required libraries

run_shell('python -m pip install requests pandas xlsxwriter xlrd openpyxl')

import sys
import os
import json
import pandas as pd
import requests as http
import datetime

HOME = os.getcwd()
if "\\" in HOME:
    slash = "\\"
else:
    slash = "/"
HOME+=slash

def now() : return datetime.datetime.today()

def logtime() : return str(now())

args = sys.argv;
OP_CODES = [ '@', '<', '?' ]

def sys_macro_write_excel_file(dataObject):
    filename = dataObject[0];
    if slash not in filename:
        filename = HOME+filename;
    mode = dataObject[1].lower();
    if mode == 'a':
        engine = 'openpyxl'
    else:
        engine = 'xlsxwriter'
    writer = pd.ExcelWriter(filename, mode=mode, engine=engine);
    write_sets = dataObject[2];
    for write_set in write_sets:
        df, sheet = write_set;
        df.to_excel(writer, sheet_name=sheet);
    writer.save();

def sys_macro_read_excel_file(dataObject):
    filename = dataObject[0];
    sheetname = dataObject[1];
    _var = dataObject[2];
    if slash not in filename:
        filename = HOME+filename;
    globals()[_var] = pd.read_excel(r'%s' % filename, sheet_name=sheetname, index_col=0);

def sys_macro_delete_file(dataObject):
    filename = dataObject[0];
    if slash not in filename:
        filename = HOME+filename;
    try:
        os.remove(filename)
    except:
        pass

def sys_macro_http_request(dataObject):
    try:
        type = dataObject[0].lower();
        if type == 'get':
            url = dataObject[1];
            try:
                _headers = { key : value for (key, value) in dataObject[2] };
            except:
                _headers = {}
            if "Content-Type" not in _headers:
                _headers["Content-Type"] = "application/json";
            _var = dataObject[3];
            resp = http.get(url, headers=_headers);
        if type == 'post':
            url = dataObject[1];
            try:
                payload = { key : value for (key, value) in dataObject[2] };
            except:
                payload = {}
            try:
                _headers = { key : value for (key, value) in dataObject[3] };
            except:
                _headers = {}
            if "Content-Type" not in _headers:
                _headers["Content-Type"] = "application/json";
            _var = dataObject[4];
            resp = http.post(url, json.dumps(payload), headers=_headers);
        if str(resp.status_code)[0] != '2':
            # non-2XX response
            print('[Percival %s console @ %s] => HTTPError: [%s] %s' % (PercivalVersion, logtime(), resp.status_code, resp.content))
        try:
            content = resp.json()
        except:
            content = resp.content
        globals()[_var] = content;
        # report
        print('[Percival %s console @ %s] => HTTPSuccess: [%s] %s bytes stored in VAR : %s' % (PercivalVersion, logtime(), resp.status_code, sys.getsizeof(globals()[_var]), _var))
    except Exception as err:
        # handle user error
        print('[Percival %s console @ %s] => HTTPUserError: %s' % (PercivalVersion, logtime(), str(err)))

SYSTEM_MACROS = {
    'READ_EXCEL' : sys_macro_read_excel_file,
    'WRITE_EXCEL' : sys_macro_write_excel_file,
    'DELETE_FILE' : sys_macro_delete_file,
    'HTTP_REQUEST' : sys_macro_http_request
}

# read input script
if len(args) > 1:
    # read Percival script
    script = args[1];
    if slash not in script:
        script = HOME+script;
    h = open(script, 'rb')
    lines = map(lambda x:x.decode(), h.readlines())
    h.close()
    # remove comments
    lines_nc = [line for line in lines if "~~" not in line]
    # join lines
    lines = "".join(lines_nc);
    # remove useless tokens
    lines = lines.replace(' ','');
    lines = lines.replace('\r','');
    lines = lines.replace('\n','');
    # ensure URL safety
    lines = lines.replace('%20', ' ');
    # read macros
    l=[]; r=[]; j=-1
    for token in lines:
        j+=1
        if token == '{':
            l.append(j)
        if token == '}':
            r.append(j)
    if len(l) != len(r):
        # parity error
        print('[Percival %s console @ %s] => ParityError: check macro boundaries' % (PercivalVersion, logtime()))
        sys.exit()
    # Execute Macros
    for _i in range(len(l)):
        _l = l[_i]; _r = r[_i];
        macro = lines[_l+1 : _r];
        # get operations in this macro
        ops = [x for x in macro.split(';') if x]
        # run operations
        for op in ops:
            # determine op_code
            op_code = None;
            for token in op:
                if token in OP_CODES:
                    op_code = token;
                    break
            if not op_code:
                # no op_code detected, skip operation
                print('[Percival %s console @ %s] => OpCodeError: no op_code detected, skipping' % (PercivalVersion, logtime()))
            else:
                # do operation
                if op_code == '<':
                    # evaluate arithmetic expression
                    _var, _expr = op.split(op_code);
                    globals()[_var] = eval(_expr);
                if op_code == '?':
                    # echo
                    _out, _var = op.split(op_code);
                    if len(_out) == 0:
                        # console log
                        print(eval(_var));
                    else:
                        # file log
                        if slash not in _out:
                            _out = HOME+_out;
                        h = open(_out, 'wb')
                        h.write(b"%s" % eval(_var))
                        h.close()
                if op_code == '@':
                    # run system macro
                    _sysMacro, _dataObject = op.split(op_code);
                    _dataObject = eval(_dataObject);
                    if _sysMacro not in SYSTEM_MACROS:
                        # invalid system macro specified
                        print('[Percival %s console @ %s] => SysMacroError: invalid system macro [%s] specified, skipping' % (PercivalVersion, logtime(), _sysMacro))
                    else:
                        # execute system macro
                        SYSTEM_MACROS[_sysMacro](_dataObject);
else:
    # no input script provided
    print('[Percival %s console @ %s] => ScriptError: no input script provided' % (PercivalVersion, logtime()))

sys.exit()

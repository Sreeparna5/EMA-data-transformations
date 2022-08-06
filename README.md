# EMA-data-transformations
EMA data column transformations, 8 to 12 channels
import pandas as pd
import sys
import re

file_name = sys.argv[1]
df = pd.read_csv(file_name)
if len(df.columns) > 82:
    print("file already converted")
else:
    print("start")
    dfs = [df.iloc[:,:1].reset_index()]
    index_tracker = 1
    last_col = 5
    if len(sys.argv) > 2:
        last_col = int(sys.argv[2])
    for i in range(1, len(df.columns), 10):
        name_dict = {}

        # index_tracker appends columns you want to put in the begining
        # processing first 4 rows
        if index_tracker in [1 ,2 ,3 ,last_col]:
            for col in df.iloc[:,i:i+10].columns:
                newcol = col
                if col.find("114") != -1:
                    newcol = col.replace("114", "112")
                    if index_tracker == last_col:
                        newcol = newcol[:len(newcol) - 7] + 'P02-CH1'
                name_dict[col] = newcol
            new_df = df.iloc[:,i:i+10].rename(index=str, columns=name_dict)
            new_df[new_df.columns[0]] = str(new_df.columns[0])
            dfs.append(new_df.reset_index(drop=True))
        index_tracker+=1

    dfs.append(df.iloc[:,1:].reset_index(drop=True))
    new_df = pd.concat(dfs,  axis=1, ignore_index=False)
    ## use any naming convention thats desirable, here new.csv will be converted to new_converted.csv
    print("converting %s" % sys.argv[1])
    new_file = file_name.replace('.csv','_converted.csv')
    ## removes 'frame.2' to 'frame'
    new_df.columns = new_df.columns.str.replace(r'\..+', '')
    new_df.iloc[:,1:].reset_index().to_csv(new_file, sep=',', encoding='utf-8', index=False)
    print("%s was created" % new_file )

## requires pandas
## pip3 install pandas
## python3 read.py filename.csv <optional_last_channel_number> -> e.g. :    python3 read.py S2000a5162019_009.csv 7 

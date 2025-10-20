<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [FileInput & FileOutput nodes](#fileinput--fileoutput-nodes)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# FileInput & FileOutput nodes

The FileInput node creates an **mqsitransitin** subdirectory in the
input directory. The mqsitransitin subdirectory holds and locks the
input files while they are being processed. The integration node reads
the file and propagates a message, or messages, by using the contents of
the file. By default the file is archived to 'mqsiarchive' under the
file input directory. If you select Move to Archive Subdirectory, the
source file is moved to the archive subdirectory of the input directory.
The subdirectory name is mqsiarchive. For example, if the input
directory is /var/fileinput, the absolute path of the archive
subdirectory is /var/fileinput/mqsiarchive. If this directory does not
exist, the integration node creates it when it first tries to move a
file there.

Specify a value for the Action on failing file property to determine
what the node is to do with the input file after all attempts to process
its contents fail: Move to Backout Subdirectory. The file is moved to
the backout subdirectory of the input directory. The name of this
subdirectory is mqsibackout. If the input directory is /var/fileinput,
the absolute path of the backout subdirectory is
/var/fileinput/mqsibackout. If this subdirectory does not exist, the
integration node creates it when it first tries to move a file
there.

FileOutput node: The file is then moved to the specified output
directory. If this transfer fails, you can specify if the file is
deleted, moved to the failure directory (mqsifailure), renamed before
being moved to the failure directory, or no action is taken on the file.

If an integration server that processes files in this input
directory is removed, check the mqsitransitin subdirectory for partially
processed or unprocessed files. Move any such files back into the input
directory and remove the integration server UUID prefix from the file
names, so that they can be processed by a different integration
server.

[← Back to Main page](../IIB_ACE.md)
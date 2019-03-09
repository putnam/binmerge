#!/usr/bin/env python3
#
# binmerge
# Takes a cue sheet with multiple binary track files and merges them together,
# generating a corrected cue sheet in the process.
#
# Please report any bugs on GitHub: https://github.com/putnam/binmerge

import argparse, re, os, subprocess

class Track:
  globalBlocksize = None

  def __init__(self, num, track_type):
    self.num = num
    self.indexes = []
    self.track_type = track_type
    self.sectors = None
    self.file_offset = None

    # All possible blocksize types. You cannot mix types on a disc, so we will use the first one we see and lock it in.
    #
    # AUDIO – Audio/Music (2352)
    # CDG – Karaoke CD+G (2448)
    # MODE1/2048 – CDROM Mode1 Data (cooked)
    # MODE1/2352 – CDROM Mode1 Data (raw)
    # MODE2/2336 – CDROM-XA Mode2 Data
    # MODE2/2352 – CDROM-XA Mode2 Data
    # CDI/2336 – CDI Mode2 Data
    # CDI/2352 – CDI Mode2 Data
    if not Track.globalBlocksize:
      if track_type in ['AUDIO', 'MODE1/2352', 'MODE2/2352', 'CDI/2352']:
        Track.globalBlocksize = 2352
      elif track_type == 'CDG':
        Track.globalBlocksize = 2448
      elif track_type == 'MODE1/2048':
        Track.globalBlocksize = 2048
      elif track_type in ['MODE2/2336', 'CDI/2336']:
        Track.globalBlocksize = 2336
      print("Locked blocksize to %d" % Track.globalBlocksize)

class File:
  def __init__(self, filename):
    self.filename = filename
    self.tracks = []
    self.size = os.path.getsize(filename)

def read_cue_file(cue_path):
  files = []
  this_track = None
  this_file = None

  f = open(cue_path, 'r')
  for line in f:
    m = re.search('FILE "?(.*?)"? BINARY', line)
    if m:
      this_file = File(os.path.join(os.path.dirname(cue_path), m.group(1)))
      files.append(this_file)

    m = re.search('TRACK (\d+) ([^\s]*)', line)
    if m:
      this_track = Track(int(m.group(1)), m.group(2))
      this_file.tracks.append(this_track)

    m = re.search('INDEX (\d+) (\d+:\d+:\d+)', line)
    if m:
      this_track.indexes.append({'id': int(m.group(1)), 'stamp': m.group(2), 'file_offset':cuestamp_to_sectors(m.group(2))})

  if len(files) == 1:
    # only 1 file, assume splitting, calc sectors of each
    next_item_offset = files[0].size // Track.globalBlocksize
    for t in reversed(files[0].tracks):
      t.sectors = next_item_offset - t.indexes[0]["file_offset"]
      next_item_offset = t.indexes[0]["file_offset"]

  for f in files:
    print("-- File --")
    print("Filename: %s" % f.filename)
    print("Size: %d" % f.size)
    print("Tracks:")

    for t in f.tracks:
      print("  -- Track --")
      print("  Num: %d" % t.num)
      print("  Type: %s" % t.track_type)
      if t.sectors: print("  Sectors: %s" % t.sectors)
      print("  Indexes: %s" % repr(t.indexes))

  return files


def sectors_to_cuestamp(sectors):
  # 75 sectors per second
  minutes = sectors / 4500
  fields = sectors % 4500
  seconds = fields / 75
  fields = sectors % 75
  return '%02d:%02d:%02d' % (minutes, seconds, fields)

def cuestamp_to_sectors(stamp):
  # 75 sectors per second
  m = re.match("(\d+):(\d+):(\d+)", stamp)
  minutes = int(m.group(1))
  seconds = int(m.group(2))
  fields = int(m.group(3))
  return fields + (seconds * 75) + (minutes * 60 * 75)


def gen_merged_cuesheet(bin_filename, files):
  cuesheet = 'FILE "%s" BINARY\n' % bin_filename
  # One sector is (BLOCKSIZE) bytes
  sector_pos = 0
  for f in files:
    for t in f.tracks:
      cuesheet += '  TRACK %02d %s\n' % (t.num, t.track_type)
      for i in t.indexes:
        cuesheet += '    INDEX %02d %s\n' % (i['id'], sectors_to_cuestamp(sector_pos + i['file_offset']))
    sector_pos += f.size / Track.globalBlocksize
  return cuesheet

def gen_split_cuesheet(bin_filename, merged_file):
  # similar to merged, could have it do both, but separate arguably cleaner
  cuesheet = ""
  for t in merged_file.tracks:
    cuesheet += 'FILE "%s (Track %02d).bin" BINARY\n' % (bin_filename, t.num)
    cuesheet += '  TRACK %02d %s\n' % (t.num, t.track_type)
    for i in t.indexes:
      sector_pos = i['file_offset'] - t.indexes[0]['file_offset']
      cuesheet += '    INDEX %02d %s\n' % (i['id'], sectors_to_cuestamp(sector_pos))
  return cuesheet

def merge_files(merged_filename, files):
  # cat is actually faster, but I prefer multi-platform and no special-casing
  chunksize = 1024 * 1024
  with open(merged_filename, 'wb') as outfile:
    for f in files:
      with open(f.filename, 'rb') as infile:
        while True:
          chunk = infile.read(chunksize)
          if not chunk:
            break
          outfile.write(chunk)
  return True

def split_files(new_basename, merged_file):
  # use calculated sectors, read the same amount, start new file when equal
  with open(merged_file.filename, 'rb') as infile:
    for t in merged_file.tracks:
      chunksize = 1024 * 1024
      out_name = '%s (Track %02d).bin' % (new_basename, t.num)
      tracksize = t.sectors * Track.globalBlocksize
      written = 0
      with open(out_name, 'wb') as outfile:
        while True:
          if chunksize + written > tracksize:
            chunksize = tracksize - written
          chunk = infile.read(chunksize)
          outfile.write(chunk)
          written += chunksize
          if written == tracksize:
            break
  return True

def main():
  parser = argparse.ArgumentParser(description="Using a cuesheet, merges numerous bin files into a single bin file and produces a new cuesheet with corrected offsets. Works great with Redump. Supports all block modes, but only binary track types. Should work on any python3 platform.")
  parser.add_argument('cuefile', help='path to source cuefile with multiple referenced bin tracks')
  parser.add_argument('new_name', help='name (without extension) for your new bin/cue files')
  parser.add_argument('--split', help='Change mode from merging to splitting to allow reconstruction of the split format.', required=False, action="store_true")
  parser.add_argument('-o', dest='outdir', required=False, default=False, help='output directory. defaults to the same directory as source cue')
  args = parser.parse_args()


  cue_map = read_cue_file(args.cuefile)
  if args.split:
    cuesheet = gen_split_cuesheet(args.new_name, cue_map[0])
  else:
    cuesheet = gen_merged_cuesheet(args.new_name+'.bin', cue_map)

  outdir = os.path.dirname(args.cuefile)
  if args.outdir:
    outdir = args.outdir

  if not os.path.exists(args.outdir):
    print("Output dir does not exist")
    return False

  with open(os.path.join(outdir, args.new_name+'.cue'), 'w', newline='\r\n') as f:
    f.write(cuesheet)
  print("Wrote %s" % args.new_name+'.cue')

  if args.split:
    print("Splitting files...")
    if split_files(os.path.join(outdir, args.new_name), cue_map[0]):
      print("Wrote %d bin files" % len(cue_map[0].tracks))
    else:
      print("Unable to split bin files")
  else:
    print("Merging files...")
    if merge_files(os.path.join(outdir, args.new_name+'.bin'), cue_map):
      print("Wrote %s" % args.new_name+'.bin')
    else:
      print("Unable to merge bin files")

main()

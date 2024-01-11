# log-compression

To update the script to uncompress `.gz` files, record the uncompressed size of each file, include a sum total of all uncompressed files, note the difference in sum total between the compressed and uncompressed file sizes, and calculate a compression ratio for individual files, you can use the following script:

```bash
#!/bin/bash

# Prompt the user for a directory path
read -p "Enter the directory path: " directory
output_file="file_sizes.txt"

# Check if the directory exists
if [ ! -d "$directory" ]; then
    echo "Error: Directory not found at $directory"
    exit 1
fi

# Initialize total size variables
total_compressed_size=0
total_uncompressed_size=0

# Header for the output file
echo "Compressed and Uncompressed File Sizes in $directory" > "$output_file"
echo "-----------------------------------------------------" >> "$output_file"

# Process .gz files
shopt -s nullglob
for file in "$directory"/*.gz; do
    # Get compressed file size in bytes (macOS compatible command)
    compressed_size=$(stat -f%z "$file")
    total_compressed_size=$((total_compressed_size + compressed_size))

    # Uncompress the file
    gunzip -c "$file" > "${file%.gz}"

    # Get uncompressed file size
    uncompressed_file="${file%.gz}"
    uncompressed_size=$(stat -f%z "$uncompressed_file")
    total_uncompressed_size=$((total_uncompressed_size + uncompressed_size))

    # Calculate compression ratio
    compression_ratio=$(echo "scale=2; $uncompressed_size / $compressed_size" | bc)

    # Write file sizes and compression ratio to output file
    echo "$(basename "$file"): Compressed size = $compressed_size bytes, Uncompressed size = $uncompressed_size bytes, Compression Ratio = $compression_ratio" >> "$output_file"
done

# Check if no .gz files were found
if [ $total_compressed_size -eq 0 ]; then
    echo "No .gz files found in $directory"
    exit 0
fi

# Calculate total difference and overall compression ratio
total_difference=$((total_uncompressed_size - total_compressed_size))
overall_compression_ratio=$(echo "scale=2; $total_uncompressed_size / $total_compressed_size" | bc)

# Write total sizes, difference, and overall compression ratio to output file
echo "-----------------------------------------------------" >> "$output_file"
echo "Total compressed size: $total_compressed_size bytes" >> "$output_file"
echo "Total uncompressed size: $total_uncompressed_size bytes" >> "$output_file"
echo "Total size difference: $total_difference bytes" >> "$output_file"
echo "Overall Compression Ratio: $overall_compression_ratio" >> "$output_file"

echo "File sizes and compression ratios recorded in $output_file"
```

This script will:

1. Uncompress each `.gz` file in the specified directory.
2. Record the size of each compressed and uncompressed file.
3. Calculate the compression ratio for each file.
4. Sum the total sizes of compressed and uncompressed files.
5. Calculate the total size difference and overall compression ratio.

The results will be saved in `file_sizes.txt`. Please note that this script will leave the uncompressed files in the directory. If you want to delete them after processing, you can add a line to remove each uncompressed file after its size is recorded.

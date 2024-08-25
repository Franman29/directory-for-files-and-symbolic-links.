# directory-for-files-and-symbolic-links.
 Bash shell script that searches the /bin directory for files and symbolic links.


#!/bin/bash

# Name: Manito Francine Beredo
# Student number: 10632689

# I constructed a script that searches the '/bin' directory for files and symbolic links. I used seven functions throughout the script.
# The first function is "exit_and_error_display", which displays an error message in red and exits the script.
# The second function is 'options_and_arguments_processed', which uses the 'getopts' command and also handles the options specified in the
# script functionality requirements. It validates the provided arguments and sets the appropriate variables.
# The third function is the 'symbolic_link_search' which searches for symbolic links within the bin directory. It scans through each item in the 
# directory and uses the '-L' to identify symbolic links. To retrieve the target of every symbolic link, I used the 'readlink -f' command and 
# the information is stored in an array which is named 'symbolic_links'. The fourth function is the 'display_symbolic_links' which the symbolic links 
# that were found is displayed during each search. It shows the number of matches in green and a display of symbolic link names and their target path which is formatted.
# The fith function is the 'matching_files_search' which looks for files that match the search criteria. It iterates through each item in the /bin directory, compares the 
# search string, and conducts byte size comparisons if enabled. If a match is detected, the file's name and size are saved in an array called matching_files.
# The sixth function is the 'files_matching_displays' which shows the matchning files that were found during the search. The display of the output depends if byte size 
# comparison is involved. Lastly, is the 'main' function which is the entry point of the script. It checks for options/arguments, processes them using 
# options_and_arguments_processed, looks for symbolic links if count_only is specified, and then looks for matching files based on criteria. Finally, depending on the 
# count_only parameter, it displays either the symbolic links or the matching files.




# Create variables for storing input and search results
string_search=""    # This variable will contain the string that we want to find
count=0     # This variable will monitor the number of matches found
found_matches=false   # This boolean variable indicates whether or not any matches were found
comparator=""   # The comparison operator that will be used for searching will be stored in this variable
operator_comparator=""  # This variable will contain the operator that will be used for value comparison
value_comparator=""    # This variable will contain the value that will be compared
count_only=false    # This boolean variable will determine whether we only want to count the matches or also display the matching files
list_symbolic_links=true    # This variable is used to see if there are symbolic links that will be listed or displayed
files_matching=()   # This array will store the paths of the files that match the search criteria
symbolic_links=()   # This array will store the paths of the symbolic links that match the search criteria



# Show an error message and then exit
function exit_and_error_display() {
    echo -e "\033[31m$1\033[0m" >&2
    exit 1
}


# Options and arguments handling and processing
function options_and_arguments_processed() {
    local count_option=0

    while getopts ":s:b:1" option; do   # 'getops' is used to determine options and arguments
        ((count_option++))        
        case $option in     # A case statement that validates the value of the variable 'option'
            s)  # The '-s' option was used
                string_search="$OPTARG"     # stored in the 'string_search' variable
                ;;
            b)  # The '-b' option was used 
                comparator_arg="$OPTARG"    # stored in the 'comparator_arg' variable
                IFS=',' read -r operator_comparator value_comparator <<< "$comparator_arg"      # Divides the value into two variables, 'operator_comparator' and 'value_comparator,' and'read -r' reads the values into the provided variables
                operator_comparator=$(echo "$operator_comparator" | tr 'a-z' 'A-Z')  # Convert to uppercase
                
                # If statement determines whether the 'comparator_operator' is one of the valid values and whether it is a numeric value
                if [[ ("$operator_comparator" = "GT" || "$operator_comparator" = "LT" || "$operator_comparator" = "GE" || "$operator_comparator" = "EQ" || "$operator_comparator" = "LE") && $value_comparator =~ ^[0-9]+$ ]] || [[ ! ("$operator_comparator" = "GT" || "$operator_comparator" = "LT" || "$operator_comparator" = "GE" || "$operator_comparator" = "EQ" || "$operator_comparator" = "LE") ]]; then
                    if [[ ! "$value_comparator" =~ ^[0-9]+$ ]]; then
                        exit_and_error_display "Invalid byte value passed. Exiting.."   # Prints error message
                    else
                        comparator="b"
                    fi
                else
                    exit_and_error_display "Invalid comparator/argument(s) passed. Exiting.."   # Prints error message
                fi
                ;;
            1)  # The '-1' option was used
                count_only=true     # set to true
                ;;
            ne)
                comparator="ne"
                ;;
            *)
                exit_and_error_display "Invalid flag or missing argument error - exiting.."     # Display an error message if an invalid option or argument is encountered
                ;;
        esac
    done

        # Check if more than one option/argument was provided
    if [ "$count_option" -gt 1 ]; then
        exit_and_error_display "Please provide only one option/argument at a time. Exiting..."
    fi

    shift "$((OPTIND-1))"   # Shifts positional parameters to remove the options and arguments that was processed by getops
}

# Search for symbolic links
function symbolic_links_search() {

    # starts with for loop that iterates over each item in the 'bin' directory. 
    for item in /bin/*; do      # The wildcard is used to match all files and directories in the 'bin' directory
        if [[ -L "$item" ]]; then   # Checks if current item is a symbolic link using '-L' operator
            target_link=$(readlink -f "$item")  # Uses 'readlink' command to get target of the symbolic link. The '-f' option is used to resolve any intermediate symbolic links in the path
            symbolic_links+=("$(basename "$item") $target_link")    # Adds basename of item (name of symbolic file) and its target to the 'symbolic_links array'
        fi
    done
}

# Display symbolic links
function display_symbolic_links() {
    echo -e "\e[32m${#symbolic_links[@]}\e[0m matches found..." # Shows how many files are matched in green
    printf "\e[34m%-30s %s\e[0m\n" "SYMBOLIC" "POINTS TO"       # Display headers for symbolic links

    # 'For loop' that iterates over each element in 'symbolic_links' array
    for link_info in "${symbolic_links[@]}"; do
        link_name=$(echo "$link_info" | awk '{print $1}')   # Used of 'awk' command to extract the link name
        link_target=$(echo "$link_info" | awk '{print $2}')     # Used of 'awk' command to extract the link target
        
        # Extract only the filename from the link target using basename
        target_filename=$(basename "$link_target")
        
        # Print the symbolic link name and the extracted target filename
        printf "%-30s %s\n" "$link_name" "$target_filename"
    done
    
    exit 0
}

# Search for matching files based on search_string and comparator
function matching_files_search() {
    for bin in /bin/*; do
        file_name="$(basename "$bin")"
        
        if [[ "$(basename "$bin")" == *"$string_search"* ]]; then  # Corrected variable name
            size=$(stat -c %s "$bin")

            if [ "$comparator" = "b" ]; then
                if [[ "$operator_comparator" = "GT" && "$size" -gt "$value_comparator" ]] ||        # Checks if size is greater than the value
                   [[ "$operator_comparator" = "LT" && "$size" -lt "$value_comparator" ]] ||        # Checks if size is less than the value
                   [[ "$operator_comparator" = "GE" && "$size" -ge "$value_comparator" ]] ||        # Checks if size is greater than or equal to the value
                   [[ "$operator_comparator" = "EQ" && "$size" -eq "$value_comparator" ]] ||        # Checks if size is equal to the value
                   [[ "$operator_comparator" = "LE" && "$size" -le "$value_comparator" ]]; then     # Checks if size is equal to the value
                    
                    file_info=""    # This variable is an empty string which stores the information about the files, which inclues the name and size
                    
                    kb_size=$(echo "scale=2; $size / 1024.0" | bc)  # Calculates the size of the file in kilobytes by diving the variable to 1024.0 
                    mb_size=$(echo "scale=2; $size / 1024 / 1024" | bc)  # Calculates the size of the file in megabytes by dividing the size variable by 1024 twice
                    mb_size_rounded=$(printf "%.2f" "$mb_size")  # Round to 2 significant figures

                    # Checks if the rounded megabytes size is not equal to '0.00'
                    if [ "$mb_size_rounded" != "0.00" ]; then
                        file_info="$(basename "$bin") ${mb_size_rounded}Mb"  # If it is not zero, it means the file size is greater than or equal to 1 megabyte
                    else
                        file_info="$(basename "$bin") ${kb_size}kb"     # If the rounded megabyte size is zero, the file_info variable is assigned the file name followed by the kilobyte size and the unit "kb"
                    fi

                    matching_files+=("$file_info")      # Add the 'file_info' variable to 'matching_files' array
                    ((count++))     # Increment the 'count' variable to 1
                    matches_found=true      # Set to true
                fi
            else
                kb_size=$(echo "scale=2; $size / 1024.0" | bc)      # Divide the file size by 1024 to get the file size in kilobytes
                mb_size=$((size / 1024 / 1024))     # Divide the file size by 1024 times to get the file size in megabytes

                file_info=""    # Create an empty string to hold the file information

                if [ "$mb_size" -gt 0 ]; then   # Determine whether the file size in megabytes is greater than zero
                    file_info="$(basename "$bin") ${mb_size}Mb"     # If the file size is more than zero, record the file name and size in megabytes in the file_info variable
                else
                    file_info="$(basename "$bin") ${kb_size}kb"     # Store the file name and size in kilobytes in the file_info variable if the file size is 0 or less
                fi

                matching_files+=("$file_info")      # Add the file_info to the array of matching_files
                ((count++))     # Increment the count variable by 1
                matches_found=true      # Set the matches_found variable to true
            fi
        fi
    done
}


# Display matching files
function files_matching_displays() {
    if [ "$found_matches" = false ] && [ "$count" -eq 0 ]; then
        echo -e "\e[34mNo matches found\e[0m"   # When no matches are discovered, this line publishes a message
    elif [ "$count_only" = true ]; then
        echo -e "\e[32m$count\e[0m matches found"   # When the count_only flag is set to true, this line reports the number of matches discovered
    else
        if [ "$comparator" = "b" ] && [ "$operator_comparator" = "EQ" ]; then
            echo -e "\e[32m$count\e[0m matches found with size equal to $value_comparator bytes:"   # When the comparator and operator_comparator variables are set appropriately, this line prints the number of matches found with a particular size
        else
            echo -e "\e[32m$count\e[0m matches found..."    # When no specific size is supplied, this line reports the number of matches discovered
        fi

        printf "\e[34m%-30s %7s\e[0m\n" "NAME" "SIZE"   # This line makes the table's header in blue, which includes the matching file names and sizes
        for file_info in "${matching_files[@]}"; do
            file_name=$(echo "$file_info" | awk '{print $1}')
            file_size=$(echo "$file_info" | awk '{print $2}')

            if [ "$comparator" = "b" ] && [ "$operator_comparator" = "LE" ]; then
                printf "%-30s %10s\n" "$file_name" "${file_size} bytes"     # When the comparator and operator_comparator variables are set appropriately, this line prints the file name and size in bytes
            else
                printf "%-30s %10s\n" "$file_name" "$file_size" # This line prints the file name and size when no specific size is specified
            fi
        done
    fi
}


# Main function
function main() {
    if [[ $# -eq 0 ]]; then     # Determines whether any options or arguments have been passed.
        exit_and_error_display "No options/arg(s) passed. Exiting..."   #If no options or arguments are given, the script will display an error message and exit
    fi

    options_and_arguments_processed "$@"  # This function called to handle the processing of options and arguments passed to the script

    symbolic_links_search   # This function is used to look for symbolic links in the directories supplied

    if [ "$count_only" = true ]; then   # If the flag 'count_only' is set to true
        display_symbolic_links      # Uses the 'display_symbolic_links' function to show the number of symbolic links discovered
    else
        matching_files_search       # If not set, it calls the 'search_matching_files' function to look for matching files based on the parameters supplied
        files_matching_displays      # Calls the display_matching_files function to display the discovered matching files
    fi
}

# Call the main function with command line arguments
main "$@"

exit 0

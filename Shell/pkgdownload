#!/bin/bash
export LANG=C 
# check for provided arguments
if [[ $# -lt 1 ]]; then
    echo "[!] No package name provided"
    exit
fi

package="$1"
filename="deb.list"
fname="complete_deb.txt"
gcount=$(apt-cache depends ${package} | grep -v "<" | grep -icw "Depends:")
it=0
masterlist="dependencies_master_$RANDOM.mlist"

# colors
black=0; red=1; green=2; yellow=3; blue=4; magenta=5; cyan=6; white=7;

### Start function declarations

# prints the banner
function print_banner {
    tput setaf 3
    echo "+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+"
    echo "|  P a c k a g e  D o w n l o a d e r  v 1.4  |"
    echo "+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+"
    echo "------------- Created by Wick3rman ------------"
    echo ""
    tput setaf 7
}

# prints a message with a given color.
function printclr {
	# $1 - Color code
	# $2 - Message
    tput setaf $1
    echo -ne "$2"
    tput sgr0
}

if [[ ${gcount} -eq 0 ]]; then
    echo "[+] Verify the package name using '$(printclr ${cyan} "apt search")' and try again"
    exit
fi

# check for internet connectivity
wget -q --tries=3 --timeout=3 --spider https://www.apple.com
if [[ $? -ne 0 ]]; then
    # Exit code not 0
    echo "$(printclr ${red} "No connection, check your internet and try again")"
    exit
fi

# reset the timer
function start_timer {
    SECONDS=0
}

# prints time elapsed since `start_timer`
function get_elapsed_time {
	duration=$SECONDS
	min=$(($duration/60))
	sec=$((duration % 60))
	[[ ${min} -eq 0 && ${sec} -gt 0 ]] && echo "$sec Seconds"
	[[ ${min} -gt 0 && ${sec} -gt 0 ]] && echo "${min}m ${sec}s"
	[[ ${min} -eq 0 && ${sec} -eq 0 ]] && echo "${sec}s"
}

# check if a dependency(package) is not present in the masterlist
function is_not_present {
    # $1 package name
    echo $(cat ${masterlist} | grep -ic "$1")
}

# add a dependecy(package) to the masterlist
function add_to_master_list {
    # $1 package name
    presr=$(is_not_present "$1")
    if [[ ${presr} -eq 0 ]]; then
        echo "$1" >> ${masterlist}
        echo "[-] Added $(printclr ${cyan} "$1") to dependency list"
    fi
}

# checks if the last command was successful
function check_if_successful {
    # $1 - Command exit code
    # $2 - Message

    if [[ $1 -eq 0 ]]; then
        # Command success
        tput cuu1
        echo "$2 $(printclr ${green} "[$(echo $'\u2714')]")"
    else
        # Command failed
        tput cuu1
        echo "$2 $(printclr ${red} "[x]")"
    fi
}

# downloads package and dependencies
function download_packages {
    # $1 - File with packages listed

    # Append the main package to file
    echo "${package}" >> $1
    add_to_master_list ${package}
    # Iterate through the list and download each dependency
    while read -r line
    do
        name="$line"
        echo "[-] Downloading: $(printclr ${cyan} "$name")"
        apt-get download "$name" &> /dev/null

        if [[ $? -eq 0 ]]; then
            # Download success
            tput cuu1
            echo "$(tput setaf 7)[-] Downloading: $(printclr ${cyan} "$name") $(printclr ${green} "[$(echo $'\u2714')]")"
        else
            # Download failed
            tput cuu1
            echo "$(tput setaf 7)[-] Downloading: $(printclr ${cyan} "$name") $(printclr ${red} "[x]")"
        fi
    done < "$1"
}

# gets dependencies and append them to package list file (deb.list)
function get_dependencies {
    # $1 package name
    p1="$1"
    apt-cache depends ${p1} | grep -v "<" | grep -w "Depends:" > "${p1}_${filename}"

    sed -i -e 's/[<>|:]//g' "${p1}_${filename}"
    sed -i -e 's/Depends//g' "${p1}_${filename}"
    sed -i -e 's/ //g' "${p1}_${filename}"

    # Local count
    lcount=$(apt-cache depends ${p1} | grep -v "<" | grep -icw "Depends:")
    lit=0

    while [ ${lit} -lt ${lcount} ]
    do
        read -r line
        name="$line"
        # Add dependency if not in the master list.
        if [[ $(is_not_present "$name") -eq 0 ]]; then
            add_to_master_list ${name}
            get_dependencies ${name}
        fi
        # lit++
        lit=$(expr ${lit} + 1)
    done < "$1_$filename"
}

# gets dependencies for a package.
function get_global {
    # $1 package name

    echo "$(tput setaf 7)[-] Found $gcount dependencies for:$(tput setaf 6) $1 $(tput setaf 7)"
    # Store all dependencies to file.
    apt-cache depends $1 | grep -v "<" | grep -w "Depends:" > "$1_$filename"
    # Clean file from unnecessary characters.
    sed -i -e 's/[<>|:]//g' "$1_$filename"
    sed -i -e 's/Depends//g' "$1_$filename"
    sed -i -e 's/ //g' "$1_$filename"

    while [ ${it} -lt ${gcount} ]
    do
        while read -r line
        do
            name="$line"
            round=$(expr ${it} + 1)

            if [[ $( echo ${name} | grep -v "<" | grep -c -w "Depends:") -lt 1 ]]; then
                if [[ $(is_not_present "$name") -eq 0 ]]; then
                    #echo "[-] Adding ${name} to masterlist"
                    add_to_master_list ${name}
                    get_dependencies ${name}
                fi
            fi
            it=$(expr ${it} + 1)
        done < "$1_$filename"
    done
}

### End function declarations

### Start main
print_banner
start_timer

# create directory with package name
mkdir "$package" &> /dev/null

if [[ $? -eq 0 ]]; then
    echo "$(printclr ${green} "[+] Created directory $(printclr ${cyan} "$package")")"
    cd "$package"

    # create masterlist file
    touch ${masterlist}

    # get dependencies for package and populate masterlist
    get_global "$package"

    # Sort and remove duplicates
    echo "" >> ${fname}
    sort *.list | uniq > ${fname}

    # read the masterlist to get child dependencies
    echo "[++] Checking for child dependencies"
    while read -r line
    do
        name="$line"
        # check if is package name
        if [[ $( echo ${name} | grep -v "<" | grep -icw "Depends:") -lt 1 ]]; then
            #echo "[$] Round $round:$(tput setaf 6) $name $(tput setaf 7)"
            pre=$(cat ${masterlist} | grep -ic "$name")

            #echo "Pret: $pre"
            if [ ${pre} -eq 0 ]; then
                get_dependencies ${name}
                add_to_master_list ${name}
            fi
        fi
        # it++
        it=$(expr ${it} + 1)
    done < "$fname"

    # delete all list files
    rm *.list

    # download packages from masterlist
    download_packages ${masterlist}

    # delete *.deb.list file
    rm ${fname}

    # package downloaded packages in a Tarball
    echo "[-] Packaging downloaded packages to $(printclr ${cyan} "$package.tar.gz")"
    cd ..

    # use tar to compress and package.
    tar -zcvf "./$package.tar.gz" "$package/" &> /dev/null

    # [Check] packaging success
    check_if_successful $? "[-] Packaging downloaded packages to $(printclr ${cyan} "$package.tar.gz")"

    echo "[-] Removing original directory"
    rm -r "./$package/"
    # [Check] if directory was removed
    check_if_successful $? "[-] Removing original directory "

    echo "$(printclr ${green} "[-] Operations completed in $(printclr ${yellow} "$(get_elapsed_time)")" )"
else
    echo "$(printclr ${red} "[!]")  Failed to create directory $(printclr ${cyan} "$package"). Delete existing directory or make sure you have sufficient permissions."
fi
### End main

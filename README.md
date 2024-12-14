#!/bin/bash

# Full Cleaner Script for Kali Linux
# Compatible with Linux kali 6.5.0-kali3-amd64

set -e

# Colors for output
YELLOW="\033[1;33m"
GREEN="\033[0;32m"
RED="\033[0;31m"
ENDCOLOR="\033[0m"

echo -e "${GREEN}[Kali-cleaner]: Starting the cleaning process...${ENDCOLOR}"

# Check if the script is run as root
if [ "$EUID" -ne 0 ]; then
    echo -e "${RED}[Kali-cleaner]: Error: Must be run as root${ENDCOLOR}"
    exit 1
fi

# Function to clean apt cache
clean_apt_cache() {
    echo -e "${YELLOW}[Kali-cleaner]: Cleaning apt cache...${ENDCOLOR}"
    sudo apt-get clean
    sudo apt-get autoclean
    echo -e "${GREEN}[Kali-cleaner]: Apt cache cleaned!${ENDCOLOR}"
}

# Function to clean temporary files
clean_temp_files() {
    echo -e "${YELLOW}[Kali-cleaner]: Cleaning temporary files...${ENDCOLOR}"
    sudo rm -rf /tmp/*
    sudo rm -rf /var/tmp/*
    echo -e "${GREEN}[Kali-cleaner]: Temporary files cleaned!${ENDCOLOR}"
}

# Function to remove old configuration files
remove_old_configs() {
    echo -e "${YELLOW}[Kali-cleaner]: Removing old configuration files...${ENDCOLOR}"
    OLDCONF=$(dpkg -l | grep "^rc" | awk '{print $2}')
    if [ -n "$OLDCONF" ]; then
        sudo apt-get purge -y $OLDCONF
        echo -e "${GREEN}[Kali-cleaner]: Old configuration files removed!${ENDCOLOR}"
    else
        echo -e "${YELLOW}[Kali-cleaner]: No old configuration files found.${ENDCOLOR}"
    fi
}

# Function to clean unused kernels
remove_old_kernels() {
    echo -e "${YELLOW}[Kali-cleaner]: Removing old kernels...${ENDCOLOR}"
    CURRENT_KERNEL=$(uname -r)
    INSTALLED_KERNELS=$(dpkg --list | grep -E 'linux-image-[0-9]+' | awk '{print $2}' | grep -v "$CURRENT_KERNEL")

    if [ -n "$INSTALLED_KERNELS" ]; then
        for KERNEL in $INSTALLED_KERNELS; do
            echo -e "${RED}[Kali-cleaner]: Removing $KERNEL...${ENDCOLOR}"
            sudo apt-get purge -y "$KERNEL"
        done
        echo -e "${GREEN}[Kali-cleaner]: Old kernels removed!${ENDCOLOR}"
    else
        echo -e "${YELLOW}[Kali-cleaner]: No old kernels to remove.${ENDCOLOR}"
    fi
}

# Function to empty trash directories
empty_trash() {
    echo -e "${YELLOW}[Kali-cleaner]: Emptying trash...${ENDCOLOR}"
    rm -rf /home/*/.local/share/Trash/*/** &> /dev/null
    rm -rf /root/.local/share/Trash/*/** &> /dev/null
    echo -e "${GREEN}[Kali-cleaner]: Trash emptied!${ENDCOLOR}"
}

# Function to remove unnecessary tools
remove_unwanted_tools() {
    echo -e "${YELLOW}[Kali-cleaner]: Removing unnecessary tools...${ENDCOLOR}"
    UNWANTED_TOOLS=(
        metasploit-framework
        commix
        veil
        sqlmap
        ghidra
        radare2
        cutter
    )

    for TOOL in "${UNWANTED_TOOLS[@]}"; do
        if dpkg -l | grep -q "$TOOL"; then
            echo -e "${RED}[Kali-cleaner]: Removing $TOOL...${ENDCOLOR}"
            sudo apt-get purge -y "$TOOL"
        else
            echo -e "${YELLOW}[Kali-cleaner]: $TOOL is not installed, skipping...${ENDCOLOR}"
        fi
    done
    echo -e "${GREEN}[Kali-cleaner]: Unnecessary tools removed!${ENDCOLOR}"
}

# Function to update the system
update_system() {
    echo -e "${YELLOW}[Kali-cleaner]: Updating and upgrading the system...${ENDCOLOR}"
    sudo apt-get update && sudo apt-get upgrade -y
    echo -e "${GREEN}[Kali-cleaner]: System updated successfully!${ENDCOLOR}"
}

# Function to display disk usage
display_disk_usage() {
    echo -e "${YELLOW}[Kali-cleaner]: Checking disk usage...${ENDCOLOR}"
    df -h | grep '^/dev/'
}

# Main menu
main_menu() {
    echo -e "\n${YELLOW}[Kali-cleaner]: Choose an action:${ENDCOLOR}"
    echo "1. Clean apt cache"
    echo "2. Clean temporary files"
    echo "3. Remove old configuration files"
    echo "4. Remove old kernels"
    echo "5. Empty trash"
    echo "6. Remove unnecessary tools"
    echo "7. Update system"
    echo "8. Display disk usage"
    echo "9. Run all actions"
    echo "10. Exit"

    read -rp "Enter your choice [1-10]: " CHOICE

    case $CHOICE in
        1) clean_apt_cache ;;
        2) clean_temp_files ;;
        3) remove_old_configs ;;
        4) remove_old_kernels ;;
        5) empty_trash ;;
        6) remove_unwanted_tools ;;
        7) update_system ;;
        8) display_disk_usage ;;
        9)
            clean_apt_cache
            clean_temp_files
            remove_old_configs
            remove_old_kernels
            empty_trash
            remove_unwanted_tools
            update_system
            ;;
        10) exit 0 ;;
        *) echo -e "${RED}[Kali-cleaner]: Invalid choice, please try again.${ENDCOLOR}" ;;
    esac
}

# Run the main menu in a loop
while true; do
    main_menu
done

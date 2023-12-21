#!/bin/bash
# vim: tabstop=8 expandtab shiftwidth=4 softtabstop=4 foldmethod=marker
# ----------------------------------------------------------------------
# Author:   DeaDSouL (Mubarak Alrashidi)
# URL:      https://unix.cafe/wp/en/2020/08/how-to-use-multiple-photo-libraries-with-digikam/
# GitLab:   https://gitlab.com/unix.cafe/digikam-multiple-libraries
# Twitter:  https://twitter.com/_DeaDSouL_
# License:  GPLv3
# ----------------------------------------------------------------------

# -------------------------------------------------------
# Configuration:
# -------------------------------------------------------

# Where do libraries live
REPO="${HOME}/Pictures/DigiKams"

# How did you install digikam
# 1 = digikam was installed via package manager like dnf
# 2 = digikam was installed via flatpak
EDITION=2

# -------------------------------------------------------
# You don't have to change anything else below this line.
# -------------------------------------------------------

# ----------------------------------------------------------------------

# Variables #{{{
# Date & Time
DATE=$(date +'%Y%m%d')
TIME=$(date +'%H%M%S')

# Script name
DIGIKAMCTL=$(basename $0)

# RC SRC
RC_FILE="digikamrc"

# RC Template
RC_TEMP="digikamrc.template"

# RC DST (1:pkg, 2:flatpak)
declare -a RC_PATH
RC_PATH[1]="${HOME}/.config/digikamrc"
RC_PATH[2]="${HOME}/.var/app/org.kde.digikam/config/digikamrc"
RC_DST="${RC_PATH[$EDITION]}"

# Temp path
_TMPDIR=/tmp/nohups/digikam

# Current path
CDIR=$(pwd)
#}}}

function bkp_rc() { #{{{
    [[ -f "${RC_DST}" && ! -L "${RC_DST}" ]] && mv -v "${RC_DST}" "${RC_DST}.bkp-${DATE}_${TIME}"
} #}}}

function used_lib() { #{{{
    [[ -f "${RC_DST}" && -L "${RC_DST}" ]] && basename $(dirname $(readlink "${RC_DST}") )
} #}}}

function use_lib() { #{{{
    if [ -z "${1}" ]; then
        echo "[ERROR]: Missing library name to activate."
        return 1
    elif [ -d "${DKLIB}" ]; then
        echo "Activating '${1}' library now.."
        # backup original digikamrc (if it was a file instead of symlink)
        bkp_rc
        # make symbolic links
        ln -svf "${DKLIB}/${RC_FILE}" "${RC_DST}"
        if [ $? == 0 ]; then
            echo "'$1' library has been successfully activated."
            return 0
        else
            echo "[ERROR]: Could not activate '$1' library."
            return 1
        fi
    else
        echo "[ERROR]: There is no library called '${1}'. You may want to create it first."
        return 1
    fi
} #}}}

function open_lib() { #{{{
    # Activating library if it wasn't
    if [ "${USEDDKL}" != "${1}" ]; then
        use_lib "${1}"
        if [ $? != 0 ]; then
            echo "[ERROR]: Since we could not activate '${1}' library, we can not open it."
            return 1
        fi
    fi
    echo "Opening '${1}' library.."
    # Digikam commands (1:pkg, 2:flatpak)
    if [ "${EDITION}" -eq 1 ]; then
        cd "${_TMPDIR}"
        nohup /usr/bin/digikam &
        cd "${CDIR}"
    elif [ "${EDITION}" -eq 2 ]; then
        cd "${_TMPDIR}"
        nohup /usr/bin/flatpak run --branch=stable --arch=x86_64 --command=digikam org.kde.digikam -qwindowtitle "${1}" &
        cd "${CDIR}"
    else
        echo '[ERROR]: Unknown Digikam edition'
        return 1
    fi
    return 0
} #}}}

function mk_lib() { #{{{
    if [ -z "${1}" ]; then
        echo "[ERROR]: Missing library name."
        return 1
    elif [ ! -f "${REPO}/${RC_TEMP}" ]; then
        echo "[ERROR]: Could not find '${RC_TEMP}'."
        echo "You need to have a copy of a digikamrc saved in: '${REPO}/${RC_TEMP}'"
        return 1
    elif [ ! -d "${DKLIB}" ]; then
        echo "Creating '${1}' library now.."
        mkdir -p "${DKLIB}/Database"
        echo -e '[Desktop Entry]\nIcon=digikam' > "${DKLIB}/.directory"
        cp -vp "${REPO}/${RC_TEMP}" "${DKLIB}/${RC_FILE}"
        sed -i "s,Database Name=.*,Database Name=${DKLIB}\/Database\/,g" "${DKLIB}/${RC_FILE}"
        sed -i "s,Database Name Face=.*,Database Name Face=${DKLIB}\/Database\/,g" "${DKLIB}/${RC_FILE}"
        sed -i "s,Database Name Similarity=.*,Database Name Similarity=${DKLIB}\/Database\/,g" "${DKLIB}/${RC_FILE}"
        sed -i "s,Database Name Thumbnails=.*,Database Name Thumbnails=${DKLIB}\/Database\/,g" "${DKLIB}/${RC_FILE}"
        echo "The '${1}' library has been successfully created.."
        return 0
    else
        echo "[ERROR]: The '${1}' library exists."
        return 1
    fi
} #}}}

function rm_lib() { #{{{
    if [ -z "${1}" ]; then
        echo "[ERROR]: Missing library name to remove."
        return 1
    elif [ "${USEDDKL}" == "${1}" ]; then
        echo "[ERROR]: Can not remove an activated library '${1}'."
        echo "[ERROR]: You need to activate another library first."
        return 1
    elif [ -d "${1}" ]; then
        echo 'We are going to remove the following library:'
        echo "${REPO}/${1}"
        while [[ "${ANSWER}" != 'y' && "${ANSWER}" != 'n' ]]; do
            read -p 'Remove it? (N/y): ' ANSWER
            [ -z "${ANSWER}" ] && ANSWER='n'
        done
        if [ "${ANSWER}" == 'y' ]; then
            echo "Removing '${1}'.."
            rm -rv "${REPO}/${1}"
            if [ $? == 0 ]; then
                echo "The '${1}' library has been successfully removed."
                return 0
            else
                echo "[ERROR]: Could not remove '${1}' library!"
                return 1
            fi
        else
            echo "Keeping '${1}' library."
            return 0
        fi
    else
        echo "[ERROR]: '${1}' doesn't exist or it's not a directory."
        return 1
    fi
} #}}}

function ls_libs() { #{{{
    echo 'Available libraries:'
    declare -a DKLIBS
    DKLIBS=$(find "${REPO}/" -maxdepth 1 -mindepth 1 -type d -exec basename "{}" \;)
    if [[ $(echo "${DKLIBS[@]}") == '' ]]; then
        echo 'There are no libraries to list. You may want to create one first'
        echo "  Type: ${DIGIKAMCTL} new myLibrary"
    else
        for DKL in ${DKLIBS[@]}; do
            if [[ "${USEDDKL}" == "${DKL}" ]]; then echo " * ${DKL}"
            else echo "   ${DKL}"; fi
        done
    fi
} #}}}

function show_usage() { #{{{
    echo "USAGE: ${DIGIKAMCTL} run <LIB> | use <LIB> | new <LIB> | rm <LIB> | ls | help"
    echo "    run <LIB>         To open a library."
    echo "                      Aliases: open."
    echo "    use <LIB>         To activate a library."
    echo "                      Aliases: activate."
    echo "    mk <LIB>          To create a library."
    echo "                      Aliases: create, new, make."
    echo "    rm <LIB>          To remove a library."
    echo "                      Aliases: remove."
    echo "    ls                To list all available libraries."
    echo "                      Aliases: list, libs."
    echo "    help              To show this menu."
    return 0
} #}}}

function init() { #{{{
    # create tmp dir
    [ ! -e  "${_TMPDIR}" ] && mkdir -p "${_TMPDIR}"
    # make sure tmp dir exists
    if [ ! -d  "${_TMPDIR}" ]; then
        echo 'Could not create the tmp directory.. exiting now'
        exit 1
    fi
    [ ! -d "${REPO}" ] && mkdir -p "${REPO}"
    pgrep '^digikam$' >/dev/null
    if [[ $? == 0 ]]; then
        echo "[ERROR]: You need to close 'DigiKam' first."
        exit 1
    fi
    DKLIB="${REPO}/${1}"
    USEDDKL=$(used_lib)
    return 0
} #}}}

function e() { #{{{
    $1 "${2}"
    exit $?
} #}}}

# ----------------------------------------------------------------------

# Main ($1: action, $2: library name)
init "${2}"
case "$1" in
    run|open)               e 'open_lib' "${2}" ;;
    use|activate)           e 'use_lib' "${2}"  ;;
    create|mk|make|new)     e 'mk_lib' "${2}"   ;;
    rm|remove)              e 'rm_lib' "${2}"   ;;
    ls|list|libs)           e 'ls_libs'         ;;
    help|h)                 e 'show_usage'      ;;
    *) echo "Try: ${DIGIKAMCTL} help"; exit 1   ;;
esac
